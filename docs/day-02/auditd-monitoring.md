# auditd — Linux Security Auditing

## Purpose

The Linux Audit system records selected security-relevant events in the kernel audit subsystem. The `auditd` daemon writes audit records to the configured audit log, commonly `/var/log/audit/audit.log`.

Day 2 focused on two targeted rules:

1. Detect writes and attribute changes to the local account database file `/etc/passwd`.
2. Record execution of the `sudo` binary.

## Installation and service setup

On Ubuntu Server:

```bash
sudo apt update
sudo apt install auditd audispd-plugins
sudo systemctl enable --now auditd
```

Verify:

```bash
sudo systemctl is-active auditd
sudo systemctl is-enabled auditd
sudo auditctl -s
```

## Persistent rules

The rules confirmed in the lab are:

```text
-w /etc/passwd -p wa -k identity_changes
-w /usr/bin/sudo -p x -k sudo_exec
```

A persistent rules file example is included at `configs/auditd/50-home-server.rules`. On Ubuntu, place a reviewed copy in:

```text
/etc/audit/rules.d/50-home-server.rules
```

Load persistent rules using the distribution's audit rules loader:

```bash
sudo augenrules --check
sudo augenrules --load
sudo auditctl -l
```

If the local system uses a different audit rules loading method, follow its configured service and rules setup rather than mixing loaders.

## Rule meaning

### Identity database watch

```text
-w /etc/passwd -p wa -k identity_changes
```

- `-w /etc/passwd`: watch the file.
- `-p wa`: record write and attribute-change access.
- `-k identity_changes`: attach a searchable key to matching events.

### Sudo execution watch

```text
-w /usr/bin/sudo -p x -k sudo_exec
```

- `-w /usr/bin/sudo`: watch the sudo executable.
- `-p x`: record execution.
- `-k sudo_exec`: attach a searchable key to matching events.

These rules provide focused visibility, not a complete audit policy. They do not capture every possible identity change or every privileged action.

## Search and inspect events

```bash
# List currently loaded rules
sudo auditctl -l

# Search events matching the identity database watch
sudo ausearch -k identity_changes

# Search sudo execution events
sudo ausearch -k sudo_exec

# Search events from the current day
sudo ausearch -ts today
```

For a readable report from matching events:

```bash
sudo ausearch -k sudo_exec --interpret
```

Audit records can include account names, command details, and other sensitive metadata. Redact output before sharing it.

## Persistence verification

The two rules were confirmed present after a server reboot:

```text
-w /etc/passwd -p wa -k identity_changes
-w /usr/bin/sudo -p x -k sudo_exec
```

The `auditd` service was also confirmed active after reboot.

## Troubleshooting

- Check daemon state: `sudo systemctl status auditd`.
- Inspect active rules: `sudo auditctl -l`.
- Inspect audit subsystem status: `sudo auditctl -s`.
- Validate/load rule files: `sudo augenrules --check` and `sudo augenrules --load`.
- Inspect daemon logs: `sudo journalctl -u auditd -b`.
- Check audit log permissions and available disk space if events are not being written.

## Operational cautions

- Audit rules can increase log volume and should be scoped intentionally.
- Do not casually disable audit rules or change immutable audit mode.
- Plan audit log rotation, retention, access control, and monitoring before production use.
- Protect audit logs because they may contain sensitive operational information.
