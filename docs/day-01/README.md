# Day 1 — Networking and SSH Foundations

## Objective

Prepare an Ubuntu home server for secure remote administration from an Arch Linux client, and understand the network and SSH steps rather than treating them as a black box.

## Lab topology

| Role | System | Operating system |
|---|---|---|
| Server | Lenovo ThinkCentre M72e | Ubuntu Server 24.04 LTS |
| Client | Acer laptop | Arch Linux |

Actual usernames, hostnames, IP addresses, and interface names are deliberately replaced with placeholders in this public documentation.

## What was completed

1. Updated the Ubuntu server's package lists and installed available updates.
2. Installed and started the OpenSSH server service.
3. Identified the network interfaces, assigned addresses, and default gateway on both machines.
4. Tested local-network reachability with `ping`.
5. Connected from the Arch client to the Ubuntu server using SSH.
6. Generated an Ed25519 key pair on the client.
7. Copied the public key to the server with `ssh-copy-id`.
8. Verified that public-key authentication allowed a remote login.
9. Investigated an intermittent host-unreachable problem caused by stale neighbour-cache information.
10. Used a targeted neighbour-cache flush on the client interface and retested connectivity.

## Important concepts learned

- An IP address identifies a host/interface at the network layer.
- A default gateway forwards traffic to destinations outside the local subnet.
- ARP/neighbour discovery maps local IP addresses to link-layer addresses.
- The SSH host key identifies the server to the client.
- The client's SSH private key proves the user's identity; the matching public key is installed on the server.
- The private key must remain secret and should be protected with a passphrase.

## Verification checklist

- [x] Server reachable over the local network.
- [x] OpenSSH server installed and running.
- [x] Client key pair generated.
- [x] Public key installed on the server.
- [x] Key-based SSH login tested.
- [x] Neighbour-cache troubleshooting understood.
- [ ] Confirm a DHCP reservation or static address for the server.
- [ ] Review and verify SSH hardening settings.
- [ ] Configure and verify the firewall.
- [ ] Verify unattended security updates.

Items left unchecked are follow-up tasks, not claims that the configuration is already complete.

## Next steps

1. Make the server's LAN address stable using a router DHCP reservation.
2. Configure SSH hardening and validate it before reloading SSH.
3. Configure UFW while preserving SSH access.
4. Verify unattended-upgrades configuration and operation.
