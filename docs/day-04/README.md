# Day 4 — Python security event monitor

## Objective

Day 4 brought the controls from Days 2 and 3 together in a lightweight Python monitor. Instead of checking each log manually, the monitor watches several local event sources and prints timestamped, severity-labelled alerts.

The monitor is a local defensive tool for the home server. It uses Python's standard library and polls the configured log files every two seconds.

## Implementation

The live script was created on the Ubuntu server at:

~~~text
/home/<server_user>/lan-monitor/security_monitor.py
~~~

A companion working note was saved as `DAY_04.md` in the same project directory. The script was compiled and run locally. The public repository contains this documentation and a source/configuration inventory; it does not claim to be a byte-for-byte copy of the private server script.

## Five monitored sources

| Source | Local log path | Purpose |
|---|---|---|
| Fail2ban | `/var/log/fail2ban.log` | Detect SSH ban and authentication-related Fail2ban events |
| auditd | `/var/log/audit/audit.log` | Surface audited system/account and privileged-command activity |
| LAN monitor | `~/lan-monitor/changes.log` | Detect newly observed, added, unknown, or changed LAN devices |
| SSH authentication | `/var/log/auth.log` | Detect failed password/public-key authentication and invalid users |
| UFW/kernel | `/var/log/kern.log` | Detect firewall block events |

The script reported all five sources as `[READY]` during verification.

## Alert rules

The monitor uses event counters held in memory with time-window tracking:

| Source/event | Initial severity | Escalation rule |
|---|---|---|
| Fail2ban `Ban ` | MEDIUM | HIGH at 3 or more bans within 600 seconds |
| SSH failed password/public-key, invalid user, authentication failure | MEDIUM | HIGH at 3 or more failures within 600 seconds |
| UFW `UFW BLOCK` | MEDIUM | HIGH at 10 or more blocks within 60 seconds |
| auditd event | MEDIUM | No additional threshold documented |
| LAN new/added/unknown/changed event | MEDIUM | No additional threshold documented |
| Other LAN change-log event | INFO | No additional threshold documented |

Alerts include a timestamp, source, severity, and message. Terminal output uses ANSI colour formatting.

## Running the monitor

The monitor was started manually on the server:

~~~bash
python3 ~/lan-monitor/security_monitor.py
~~~

Check syntax before running:

~~~bash
python3 -m py_compile ~/lan-monitor/security_monitor.py
~~~

The monitor starts reading at the current end of each log (tail-style); it does not replay old log history. It polls every two seconds. It was not configured to start automatically after reboot during this stage.

## Verification performed

- Python syntax compilation completed successfully.
- All five configured log sources reported READY.
- A live audit event was detected.
- A LAN test event was detected from the LAN change log.
- A synthetic Fail2ban event sequence produced MEDIUM, MEDIUM, then HIGH as the threshold was reached.
- Three SSH authentication failures during testing produced MEDIUM, MEDIUM, then HIGH.
- A firewall/Nmap test produced a UFW event that the monitor detected.

Synthetic tests are identified as synthetic; they are not evidence that a real attacker or real Fail2ban ban occurred.

## Limitations

- The monitor must be started manually; automatic startup was not configured.
- Correlation counters are in memory and reset when the process restarts.
- Alerts are printed to the terminal; no persistent alert database or notification channel was added.
- Log parsing is based on text patterns and should be hardened against format changes and log rotation.
- Access to protected logs may require appropriate group membership or elevated privileges. Do not solve this by making sensitive logs world-readable.

## Security and privacy

This public repository uses placeholders for the server account and does not include raw authentication/audit logs, LAN inventories, or real IP addresses. Never publish log lines that reveal usernames, public IPs, MAC addresses, hostnames, or other private details.

## Next improvements

1. Add robust handling for log rotation, truncation, and temporary file absence.
2. Persist alert events and correlation state if restart continuity is required.
3. Add structured parsing rather than relying only on text matching.
4. Configure a least-privilege startup method only after reviewing log access and service permissions.
5. Add tests for malformed lines, repeated events, and threshold-window expiry.
