# Day 5 — Live security detection and reboot verification

## Objective

Day 5 brought the home-server security controls together and tested whether the monitoring workflow could detect controlled security events. The focus was on live SSH authentication failures, firewall events generated during Nmap testing, and persistence of core server protections after reboot.

## Live detection tests

### 1. SSH authentication-failure detection

From the authorized Arch Linux client, three deliberately invalid SSH authentication attempts were made against the lab server using a non-existent test account and public-key-only authentication:

~~~bash
for i in 1 2 3; do
  ssh -o BatchMode=yes \
      -o PreferredAuthentications=publickey \
      -o PubkeyAuthentication=no \
      -o ConnectTimeout=5 \
      invalid-lab-user@<server_ip>
done
~~~

The client received `Permission denied (publickey)` for each attempt. The Python monitor detected the corresponding SSH authentication failures and displayed:

1. MEDIUM
2. MEDIUM
3. HIGH

This was a controlled test against the user's own lab server, not evidence of an external attack.

### 2. Firewall event detection during Nmap testing

An authorized Nmap scan was run against the lab environment. The resulting UFW firewall log event was detected by the Python monitor. This confirmed the path from firewall logging to the monitor's event display.

Only scan systems and networks that you own or are explicitly authorized to test.

### 3. Monitor source readiness

The monitor reported all five configured sources as READY:

- Fail2ban
- auditd
- LAN change log
- SSH authentication log
- UFW/kernel log

The monitor remained a manually started foreground process; automatic startup was not configured.

## Reboot and persistence verification

After reboot, the following core controls were checked:

| Component | Observed result |
|---|---|
| UFW | Active; OpenSSH allowed |
| SSH service | Active |
| auditd service | Active |
| Fail2ban service | Active |
| audit rule for `/etc/passwd` | Loaded (`identity_changes`) |
| audit rule for `/usr/bin/sudo` | Loaded (`sudo_exec`) |

Fail2ban showed three total failures and zero currently banned at one check. After reboot, its runtime counters were observed reset to zero. Runtime counters resetting is distinct from the service and jail configuration failing to persist; the service remained active.

## Scheduled LAN monitor evidence

The hourly cron schedule was checked, and a run was observed during the 15:00–15:05 window. The local monitoring workspace contained:

- `baseline.tsv`
- `previous.tsv`
- `previous-ports.tsv`

The earlier Day 3 documentation records an unresolved Nmap XML extraction issue. A genuine device/port change detected end-to-end by a scheduled run was not confirmed, so scheduled change detection is not marked fully verified.

## Project status

The core learning and lab exercises for Days 1–5 were completed and documented:

- Networking and SSH key authentication.
- Fail2ban and auditd host protection/auditing.
- Nmap discovery and tcpdump packet inspection.
- Python event monitoring across five local log sources.
- Controlled SSH and firewall event detection, plus reboot checks for core services.

This is a learning lab, not a claim of production-grade security or a complete penetration test.

## Remaining improvements

1. Fix and test Nmap XML parsing before trusting automated host/port change comparisons.
2. Demonstrate a real host/port change detected by a scheduled cron execution.
3. Verify the exact `unattended-upgrades` periodic configuration and its logs; package/service installation alone is not sufficient proof of automatic security updates.
4. Add robust log rotation/truncation handling to the Python monitor.
5. Consider durable alert storage and a least-privilege startup service if continuous monitoring is desired.
6. Add tests for parser errors, missing logs, and correlation-window expiry.

## Privacy and safety

This public documentation uses placeholders for IP addresses and account names. Do not commit raw authentication/audit logs, PCAP files, private keys, passwords, real LAN inventories, or alert output containing identifying details.
