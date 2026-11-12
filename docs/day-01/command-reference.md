# Day 1 Command Reference

Commands below are grouped by purpose. Run server-side commands on Ubuntu unless otherwise noted.

## Package management (Ubuntu)

| Command | Purpose |
|---|---|
| `sudo apt update` | Refresh package indexes. |
| `sudo apt upgrade` | Install available package upgrades. |
| `sudo apt install openssh-server` | Install the SSH server package. |

## Service management (Ubuntu)

| Command | Purpose |
|---|---|
| `sudo systemctl enable --now ssh` | Enable SSH at boot and start it now. |
| `sudo systemctl status ssh` | Inspect SSH service status. |
| `sudo systemctl reload ssh` | Reload SSH configuration without intentionally stopping the service. |
| `sudo ss -tlnp | grep ':22'` | Check for a listener on TCP port 22. |

## Network inspection (client or server)

| Command | Purpose |
|---|---|
| `ip -br address` | Show concise interface/address information. |
| `ip route` | Show routing table and default route. |
| `ip link` | Show network interfaces and link state. |
| `ip neigh` | Show neighbour/ARP cache entries. |
| `ping -c 4 <server_ip>` | Send four ICMP echo requests. |
| `sudo ip neigh flush dev <client_interface>` | Flush cached neighbour entries for one interface. |

## SSH client and key management (client)

| Command | Purpose |
|---|---|
| `ssh <server_user>@<server_ip>` | Connect to the server. |
| `ssh -v <server_user>@<server_ip>` | Connect with verbose diagnostics. |
| `ssh-keygen -t ed25519 -C "home-server-client"` | Generate an Ed25519 key pair. |
| `ssh-copy-id -i ~/.ssh/id_ed25519.pub <server_user>@<server_ip>` | Install the public key on the server. |
| `chmod 700 ~/.ssh` | Restrict access to the SSH directory. |
| `chmod 600 ~/.ssh/id_ed25519` | Restrict access to the private key. |
| `chmod 644 ~/.ssh/id_ed25519.pub` | Set normal readable permissions on the public key. |

## SSH server validation (Ubuntu server)

| Command | Purpose |
|---|---|
| `sudo sshd -t` | Check SSH daemon configuration syntax. |
| `sudo sshd -T` | Print effective SSH daemon configuration. |
| `sudo systemctl reload ssh` | Apply validated configuration changes. |

## Firewall baseline — apply only after planning access

A basic UFW policy for a server administered over SSH:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose
```

Before enabling UFW remotely, confirm that the SSH allow rule matches the actual SSH port and intended source network. Keep an existing SSH session open and test a second connection.

## Automatic security updates — verify, do not assume

Install the package if it is part of the planned setup:

```bash
sudo apt install unattended-upgrades
```

Inspect its service/timer state and periodic configuration:

```bash
systemctl status unattended-upgrades
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Package installation alone does not prove that automatic updates are configured as intended.

## Command safety notes

- Replace placeholders such as `<server_ip>` with values from your own lab.
- Do not run commands copied from documentation without understanding their target.
- Avoid broad network configuration changes over SSH.
- Never use `ip addr flush` as a substitute for `ip neigh flush`.
- Redact IP addresses, usernames, hostnames, and sensitive log content before publishing output.
