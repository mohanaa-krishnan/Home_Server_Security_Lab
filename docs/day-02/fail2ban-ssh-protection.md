# Fail2ban — SSH Brute-Force Protection

## Purpose

Fail2ban monitors configured log sources for repeated authentication failures. When a jail's threshold is reached, its action temporarily blocks the offending source address.

For this lab, the protected service is OpenSSH.

## Installation and service setup

On Ubuntu Server:

```bash
sudo apt update
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
```

Verify installation and service state:

```bash
fail2ban-client --version
sudo systemctl is-active fail2ban
sudo systemctl is-enabled fail2ban
```

## SSH jail policy

The lab's SSH jail policy uses these values:

| Setting | Value | Meaning |
|---|---:|---|
| `enabled` | `true` | Enable the SSH jail |
| `maxretry` | `4` | Maximum failures before a ban |
| `findtime` | `10m` | Window in which failures are counted |
| `bantime` | `5m` | Temporary ban duration |

A portable example is provided in `configs/fail2ban/sshd.local`. On Ubuntu, a local override can be installed at:

```text
/etc/fail2ban/jail.d/sshd.local
```

Review existing Fail2ban configuration before deploying an override. Do not edit packaged defaults in `jail.conf`; local overrides are easier to maintain.

After placing or changing a jail file:

```bash
sudo fail2ban-client -t
sudo systemctl restart fail2ban
```

If the configuration test fails, do not restart blindly; inspect the reported file and correct the syntax first.

## Verify the SSH jail

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

The jail status reports the filter/action state, failure counters, and currently banned addresses. The exact counters change over time.

Useful log inspection:

```bash
sudo journalctl -u fail2ban --since "today"
sudo tail -n 100 /var/log/fail2ban.log
```

The log file may not exist on every installation if logging is configured differently; the systemd journal is another useful source.

## Day 2 verification

After reboot, the service was confirmed active and enabled, and the `sshd` jail was present. At the recorded post-reboot check:

- Currently failed: 0
- Total failed: 3
- Currently banned: 0
- Total banned: 0

These are historical observations from that check, not fixed expected values.

## Troubleshooting checklist

1. Check service state: `sudo systemctl status fail2ban`.
2. Check configuration syntax: `sudo fail2ban-client -t`.
3. Confirm the jail is enabled: `sudo fail2ban-client status`.
4. Inspect the jail: `sudo fail2ban-client status sshd`.
5. Review service logs: `sudo journalctl -u fail2ban -b`.
6. Confirm the SSH authentication log source is available and matches the jail filter.

Do not test bans by repeatedly failing authentication from an important remote administration address. A ban can interrupt your own access.

## Operational cautions

- Keep key-based SSH access working before testing.
- Keep a second administrative session open during remote security changes.
- A short ban time is suitable for a learning lab, but production policy should be selected based on risk and operational needs.
- Fail2ban is not a substitute for strong authentication, patching, or firewall policy.
