# Detection-as-Code Lab

A hands-on security lab for building, testing, and version-controlling detections
against a simulated multi-host environment. The goal is not just to detect attacks,
but to treat detections like software: version-controlled, tested, and continuously
validated through CI.

## Why this project

Most detection work fails on consistency and validation — rules get written once,
never tested, and silently stop working. This lab practices the detection
engineering lifecycle end to end: simulate an attack, collect telemetry, write a
detection, and prove it fires before it ships.

## Architecture

Three ARM64 Ubuntu VMs running locally on UTM (Apple Silicon), on an isolated lab network.

| Host     | IP             | Role                                             |
|----------|----------------|--------------------------------------------------|
| victim   | 192.168.64.10  | Monitored endpoint (auditd + Wazuh agent)        |
| splunk   | 192.168.64.20  | SIEM — Wazuh manager (collects logs, runs rules) |
| attacker | 192.168.64.30  | Attack host (simulates adversary activity)       |

Data flow: attacker generates activity on victim -> victim records it with
auditd / sshd -> the Wazuh agent ships logs to the manager -> the manager matches
detection rules and raises alerts.

## Telemetry sources (victim)

- auditd — identity file changes (/etc/passwd, /etc/shadow), privilege
  escalation (/etc/sudoers), persistence (cron), and root command execution.
- sshd / journald — authentication events (login success/failure).

## Detection coverage (MITRE ATT&CK)

| Technique                       | ATT&CK ID | Telemetry | Status   |
|---------------------------------|-----------|-----------|----------|
| Brute force / password guessing | T1110     | sshd      | Detected |
| Privilege escalation (sudoers)  | T1548     | auditd    | Planned  |
| Persistence (cron)              | T1053.003 | auditd    | Planned  |
| Reverse shell                   | T1059.004 | auditd    | Planned  |

## Status

- [x] Multi-VM lab built, networked, and reachable
- [x] Wazuh manager deployed (SIEM)
- [x] Wazuh agent on victim, shipping logs to the manager
- [x] auditd monitoring configured on victim
- [x] First detection validated — SSH brute-force attempt raised an alert
- [ ] Attack simulations collected as samples
- [ ] Detections written as code (Sigma)
- [ ] Positive/negative tests per rule
- [ ] CI pipeline validating detections on every change

## Repo layout

- detections/  detection rules (Sigma)
- tests/       test samples per rule (positive + negative)
- infra/       environment setup notes / scripts
- docs/        architecture notes and screenshots

## Notes

The lab originally targeted a cloud build on Azure; free-tier regional and vCPU
quota limits made a reliable multi-VM deployment impractical, so it was moved to
local virtualization. This mirrors a real constraint — planning around cloud quota
and capacity — and kept the detection work fully reproducible offline.
