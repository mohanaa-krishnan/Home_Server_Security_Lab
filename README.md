# Home Server Security Lab

A practical home-lab project for learning Linux server administration, networking, SSH security, firewall configuration, automated updates, and basic security monitoring.

## Lab overview

- **Server:** Lenovo ThinkCentre-class machine running Ubuntu Server 24.04 LTS
- **Client:** Arch Linux laptop
- **Remote access:** OpenSSH over the local network
- **Focus:** Learn each security control, verify its behavior, and document the commands and troubleshooting steps.

> **Privacy:** This public repository intentionally uses placeholders for usernames, hostnames, IP addresses, and network-interface names. Never commit private keys, passwords, tokens, or unredacted logs.

## Day 1 — Networking and SSH foundations

- Updated the Ubuntu server and installed/enabled OpenSSH.
- Identified client/server network interfaces, addresses, and the default gateway.
- Tested LAN reachability and connected to the server over SSH.
- Generated an Ed25519 SSH key pair on the client.
- Installed the public key on the server with `ssh-copy-id`.
- Verified key-based login.
- Investigated stale neighbour-cache entries and used a targeted neighbour-cache flush to restore connectivity.
- Documented the next hardening steps without assuming that unverified settings were complete.

### Day 1 documentation

- [Day 1 overview](docs/day-01/README.md)
- [Networking and SSH foundations](docs/day-01/networking-and-ssh-foundations.md)
- [SSH key authentication and hardening](docs/day-01/ssh-key-authentication.md)
- [Command reference](docs/day-01/command-reference.md)

## Learning roadmap

1. **Day 1:** Networking, OpenSSH, and key-based authentication.
2. **Next:** SSH hardening, firewall rules, and automatic security updates.
3. **Later:** Intrusion-prevention and auditing tools, network discovery, packet inspection, and a small security-event monitor.

## Safe lab practices

- Use a non-root account for administration.
- Keep a working SSH session open while changing SSH configuration.
- Validate SSH configuration before reloading the service.
- Allow SSH through the firewall before enabling a restrictive incoming policy.
- Verify configuration and service status rather than relying only on installation success.
