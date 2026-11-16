# Day 4 — Verification record and known limitations

## Tests recorded

| Test | Result recorded | Evidence classification |
|---|---|---|
| Python compile check | Passed using `python3 -m py_compile` | Direct command verification |
| Five log sources | All five reported `[READY]` | Direct runtime verification |
| Audit event | A live audit event was detected | Live test |
| LAN event | A LAN test event was detected | Test event in the change log |
| Fail2ban escalation | MEDIUM, MEDIUM, HIGH | Synthetic event sequence; not a real ban |
| SSH failure escalation | MEDIUM, MEDIUM, HIGH after three failed attempts | Live authentication test |
| Firewall event | UFW block generated during an Nmap firewall test was detected | Live lab test |

The recorded SSH test involved three failed authentication attempts against the lab server. A test failure is not evidence of an external compromise.

## What was not completed

- Automatic startup after reboot was not configured.
- No persistent alert database was added.
- Correlation counters reset when the process exits.
- No notification integration was added.
- Robust log-rotation and structured parsing improvements remained future work.

## Operational commands

Compile:
~~~bash
python3 -m py_compile ~/lan-monitor/security_monitor.py
~~~

Run manually:
~~~bash
python3 ~/lan-monitor/security_monitor.py
~~~

Stop the foreground monitor with Ctrl+C.

## Safe operation

The monitor reads security-sensitive logs. Keep file permissions restrictive and grant only the access needed for the account running the monitor. Avoid running a long-lived monitoring process as root unless there is a deliberate, documented reason and an appropriate service hardening plan.
