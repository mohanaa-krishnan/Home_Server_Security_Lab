# Day 2 — Intrusion Prevention and System Auditing

## Objective

Add two complementary defensive controls to the Ubuntu home server:

- **Fail2ban** watches authentication logs and temporarily blocks source addresses that repeatedly fail SSH authentication.
- **auditd** records selected security-relevant activity so changes and privileged command execution can be reviewed.

Fail2ban is a response control; auditd is an accountability and investigation control. Neither replaces secure SSH configuration, firewall rules, patching, or careful account management.

## Lab context

- Server: Lenovo ThinkCentre M72e
- Operating system: Ubuntu Server 24.04 LTS
- Administration: OpenSSH from an Arch Linux client

This public repository uses placeholders and omits private LAN details.

## Work completed

### 1. Fail2ban

- Installed the `fail2ban` package.
- Enabled and started the service.
- Confirmed the `sshd` jail was loaded and active.
- Configured the SSH jail with:
  - `maxretry = 4`
  - `findtime = 10m`
  - `bantime = 5m`
- Checked the jail's counters and ban state.
- Verified after a server reboot that Fail2ban and the SSH jail were active.

The post-reboot status check showed zero failures currently counted in the window, three total failures recorded, and zero currently/ever banned addresses at the time of that check. These counters are a point-in-time observation, not a guarantee that every attack will be blocked.

### 2. auditd

- Installed and enabled the Linux Audit daemon.
- Added persistent rules to watch:
  - Writes and attribute changes to `/etc/passwd`, tagged `identity_changes`.
  - Execution of `/usr/bin/sudo`, tagged `sudo_exec`.
- Loaded the rules and inspected the active rule list.
- Rebooted the server and confirmed that both rules were loaded again.

Confirmed rules:

```text
-w /etc/passwd -p wa -k identity_changes
-w /usr/bin/sudo -p x -k sudo_exec
```

### 3. Reboot verification

After reboot, the following services were confirmed active:

- `ssh`
- `fail2ban`
- `auditd`

The UFW firewall was also confirmed active, with the OpenSSH application rule allowed.

## Reproduce or inspect the configuration

The repository includes example configuration files:

- [Fail2ban SSH jail](../../configs/fail2ban/sshd.local)
- [Persistent audit rules](../../configs/auditd/50-home-server.rules)

These are portable examples of the settings used in the lab. Review the target server's existing configuration before copying them; do not overwrite unrelated local policy.

## Verification commands

```bash
# Fail2ban service and jail
sudo systemctl is-active fail2ban
sudo systemctl is-enabled fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Audit daemon and active rules
sudo systemctl is-active auditd
sudo systemctl is-enabled auditd
sudo auditctl -l

# Search audit events by rule key
sudo ausearch -k identity_changes
sudo ausearch -k sudo_exec

# Check other security services
sudo systemctl is-active ssh
sudo ufw status verbose
```

## Security notes

- Fail2ban bans are temporary and depend on the configured jail, log source, and firewall action.
- Do not deliberately generate repeated failed logins against a production host. Test only in your own lab and keep an existing administrative session available.
- Audit rules can generate significant event volume on busy systems. Start with narrowly scoped rules and review disk usage and audit log retention.
- The `/etc/passwd` watch detects writes and attribute changes; it is not a complete identity-management audit. Consider additional rules for `/etc/shadow`, `/etc/group`, SSH configuration, and privileged policy only after evaluating noise and operational impact.
- Audit logs may contain usernames, command details, and other sensitive information. Do not publish raw logs.

## Day 2 completion checklist

- [x] Install and enable Fail2ban.
- [x] Confirm the SSH jail is active.
- [x] Configure retry threshold and ban window.
- [x] Install and enable auditd.
- [x] Add persistent passwd and sudo audit rules.
- [x] Confirm audit rules are loaded.
- [x] Verify service and rule persistence after reboot.
- [x] Confirm UFW remains active and OpenSSH remains allowed.

## Next steps

- Expand audit coverage carefully and document the reason for each rule.
- Review Fail2ban logs and audit events during controlled lab tests.
- Continue with network discovery and packet inspection in the next lab stage.
