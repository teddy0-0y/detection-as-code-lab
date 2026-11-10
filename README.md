# Detection-as-Code Lab

A hands-on security lab for building, testing, and version-controlling detections against a simulated multi-host environment. Detections are treated like software: version-controlled, tested, and continuously validated.

## Architecture

Three ARM64 Ubuntu VMs running locally on UTM (Apple Silicon), on an isolated lab network.

| Host     | IP             | Role                                             |
|----------|----------------|--------------------------------------------------|
| victim   | 192.168.64.10  | Monitored endpoint (auditd + Wazuh agent)        |
| splunk   | 192.168.64.20  | SIEM — Wazuh manager (collects logs, runs rules) |
| attacker | 192.168.64.30  | Attack host (simulates adversary activity)       |

Data flow: attacker generates activity on victim -> victim records it with auditd / sshd -> the Wazuh agent ships logs to the manager -> the manager matches detection rules and raises alerts.

## Detection coverage (MITRE ATT&CK)

| Technique                       | ATT&CK ID  | Status   |
|---------------------------------|------------|----------|
| Brute force / password guessing | T1110      | Detected |
| Backdoor account creation       | T1136      | Detected |
| Privilege escalation (sudoers)  | T1548.003  | Detected |
| Persistence (cron)              | T1053.003  | Detected |
| Reverse shell                   | T1059.004  | Detected |

## Detection gaps found and closed

Running the attacks exposed three blind spots — the most valuable output of the lab:

1. sudoers.d bypass — the initial rule watched only /etc/sudoers. Writing to /etc/sudoers.d/ was not caught by the file-watch rule. Fixed by watching the whole /etc/sudoers.d/ directory.
2. cron.d bypass — watching only /etc/crontab missed persistence dropped into /etc/cron.d/. Fixed by watching the directory.
3. Reverse shell, fully silent — a bash reverse shell touches none of the monitored files and was not caught by the execve rule. Fixed with a behavior-based rule on bash issuing outbound connect() syscalls.

Each gap, its root cause, and the fix is encoded as a Sigma rule in detections/.

## Repo layout

- detections/   detection rules (Sigma)
- tests/        test samples per rule (in progress)
- infra/        environment setup notes
- docs/         architecture notes and screenshots

## Notes

The lab originally targeted a cloud build on Azure; free-tier regional and vCPU quota limits made a reliable multi-VM deployment impractical, so it was moved to local virtualization.
