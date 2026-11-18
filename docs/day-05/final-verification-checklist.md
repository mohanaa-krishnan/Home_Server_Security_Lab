# Day 5 — Final verification checklist

Use this checklist when repeating the final checks. Run commands on the Ubuntu server unless a client-side section says otherwise.

## Core services after reboot

~~~bash
sudo systemctl is-active ssh
sudo systemctl is-active fail2ban
sudo systemctl is-active auditd
sudo ufw status verbose
~~~

Expected observations from the recorded lab verification:
- SSH, Fail2ban, and auditd reported active.
- UFW reported active with the OpenSSH rule allowed.

## Fail2ban

~~~bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
~~~

Interpret current counters carefully. A reboot can reset runtime failure counters; that does not by itself mean the jail configuration was lost.

## auditd rules

~~~bash
sudo auditctl -l
sudo ausearch -k identity_changes -i
sudo ausearch -k sudo_exec -i
~~~

The recorded persistent rules watched the `/etc/passwd` identity-related changes and execution of `/usr/bin/sudo`.

## Python monitor

~~~bash
python3 -m py_compile ~/lan-monitor/security_monitor.py
python3 ~/lan-monitor/security_monitor.py
~~~

The monitor was manually started and reported five sources READY. Stop it with Ctrl+C. It was not configured for automatic startup.

## Controlled SSH failure test

Perform only against your own lab server, and ensure you have a valid recovery login path before testing:

~~~bash
for i in 1 2 3; do
  ssh -o BatchMode=yes \
      -o PreferredAuthentications=publickey \
      -o PubkeyAuthentication=no \
      -o ConnectTimeout=5 \
      invalid-lab-user@<server_ip>
done
~~~

The recorded test produced three `Permission denied (publickey)` client responses and monitor severities MEDIUM, MEDIUM, HIGH.

Avoid repeated attempts beyond the planned test. Fail2ban may temporarily ban the test client depending on the active jail policy.

## Firewall and Nmap test

Use Nmap only against systems/networks you own or have explicit permission to test. The recorded authorized scan generated a UFW log event that the monitor detected.

## LAN cron monitor

~~~bash
crontab -l
tail -n 100 ~/lan-monitor/cron.log
ls -l ~/lan-monitor/baseline.tsv ~/lan-monitor/previous.tsv ~/lan-monitor/previous-ports.tsv
~~~

The hourly job was observed running during a 15:00–15:05 window and the three state files existed. A controlled change detected by a scheduled run was not confirmed. Also fix the previously recorded XML extraction issue before relying on comparisons.

## Automatic updates — verification still required

Do not mark unattended upgrades complete based only on package installation. Inspect the effective periodic settings and logs:

~~~bash
cat /etc/apt/apt.conf.d/20auto-upgrades
systemctl status apt-daily.timer apt-daily-upgrade.timer --no-pager
sudo journalctl -u apt-daily-upgrade.service --since "7 days ago" --no-pager
~~~

Review the output and confirm the intended security-update policy for the server.

## Safety

- Keep an existing administrative SSH session open while changing SSH/firewall settings.
- Do not expose sensitive logs with world-readable permissions.
- Redact account names, addresses, MAC addresses, hostnames, and tokens before sharing output.
- Do not commit private keys, raw logs, packet captures, or real network inventories.
