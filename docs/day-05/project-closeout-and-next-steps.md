# Home Server Security Lab — Project closeout

## What the five-day lab covered

| Day | Focus | Main outcome |
|---|---|---|
| 1 | Networking and SSH | Remote access using Ed25519 key authentication; LAN troubleshooting |
| 2 | Host protection and auditing | Fail2ban SSH jail, auditd rules, UFW and reboot checks |
| 3 | Network visibility | Nmap discovery, hourly scan scheduling, tcpdump inspection |
| 4 | Security event monitoring | Python monitor reading five local event sources |
| 5 | Integration and verification | Controlled SSH-failure and firewall-event detection; core service persistence checks |

## What the project demonstrates

- Linux server administration on Ubuntu and an Arch Linux client.
- SSH key-based authentication and service hardening concepts.
- Firewall configuration and intrusion-prevention fundamentals.
- Linux audit logging and event investigation.
- Authorized network discovery and packet capture.
- Python log monitoring, threshold-based correlation, and test-driven verification.
- Documentation that distinguishes observed results from unfinished work.

## Known gaps at closeout

- Nmap XML extraction had an unresolved parsing issue; automated LAN change detection needs a reliable parser and an end-to-end scheduled change test.
- The Python monitor runs manually and has no automatic startup, persistent alert database, or notification integration.
- Exact unattended-upgrades periodic configuration and successful security-upgrade evidence still need verification.
- Log rotation, structured parsing, and resilient error handling can be improved.

## Evidence discipline

A service showing active is not the same as proving every policy is correct. A configured cron schedule is not the same as proving a scheduled change was detected. A synthetic event is not a real intrusion. Keep these distinctions clear in project reports and interviews.

## Suggested resume description

**Home Server Security Monitoring Lab (Ubuntu, Arch Linux, Python, Nmap, tcpdump, Fail2ban, auditd, UFW)**

Built a self-managed Linux security lab covering SSH key authentication, host firewalling, brute-force protection, audit logging, LAN discovery, packet inspection, and a Python log-monitoring prototype. Validated controlled SSH authentication failures and firewall events, observed severity escalation, and checked core security services and audit rules after reboot. Documented remaining reliability gaps, including Nmap XML parsing and persistent monitor deployment.

Adjust the description to match the work you can demonstrate live. Do not claim production deployment, automated incident response, or fully reliable scheduled LAN change detection.
