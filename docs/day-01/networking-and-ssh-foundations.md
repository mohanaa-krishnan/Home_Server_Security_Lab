# Networking and SSH Foundations

## 1. Understand the lab network

The client and server communicate over the local network. Before troubleshooting SSH, identify each machine's interface, address, and route.

Run on Linux:

```bash
ip -br address
ip route
ip link
```

- `ip -br address` gives a concise view of interfaces and assigned addresses.
- `ip route` shows routing information, including the default gateway.
- `ip link` lists network interfaces and their link state.

Use the interface that is actually connected to the LAN. Do not assume that an interface name is the same across machines.

## 2. Test basic connectivity

From the client:

```bash
ping -c 4 <server_ip>
```

If this fails, check the following before debugging SSH:

1. Are both machines connected to the expected network?
2. Does each machine have a valid address?
3. Is the subnet and route correct?
4. Is the server online?
5. Is a firewall blocking ICMP or other traffic?

A failed ping does not by itself prove that SSH is unavailable; ICMP may be filtered.

## 3. Install and start OpenSSH on Ubuntu Server

On the server:

```bash
sudo apt update
sudo apt upgrade
sudo apt install openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

Check whether SSH is listening:

```bash
sudo ss -tlnp | grep ':22'
```

The usual SSH TCP port is 22, unless the server has been configured to use another port.

## 4. Connect from the client

On the client:

```bash
ssh <server_user>@<server_ip>
```

On the first connection, SSH may ask you to verify the server's host-key fingerprint. Verify the fingerprint through a trusted channel before accepting it. This protects against connecting to an unexpected server.

For troubleshooting:

```bash
ssh -v <server_user>@<server_ip>
```

Verbose mode provides useful connection and authentication details. Do not publish logs without checking them for private information.

## 5. Key networking concepts

- **IP address:** Logical network-layer address assigned to an interface.
- **Default gateway:** Router used for destinations outside the directly connected network.
- **MAC address:** Link-layer address used for local-network delivery.
- **ARP/neighbour table:** Cache mapping local IP addresses to link-layer addresses.
- **Host key:** Server identity key checked by SSH clients.
- **User authentication key:** Client-held private key used to authenticate a user to the server.

## 6. Neighbour-cache troubleshooting

Inspect the neighbour table:

```bash
ip neigh
```

A stale or failed neighbour entry can interfere with local-network communication. If you have confirmed the correct client interface, remove only its cached neighbour entries:

```bash
sudo ip neigh flush dev <client_interface>
```

Then retry:

```bash
ping -c 4 <server_ip>
ssh <server_user>@<server_ip>
```

A neighbour-cache flush removes cached mappings; it does not change the machine's IP configuration. Do not confuse it with commands that flush interface addresses.

## 7. Keep the server address stable

A DHCP reservation on the router is often the simplest home-lab option. It makes the server receive the same LAN address while keeping address management centralized. If you configure a static address directly on the server, ensure the address, prefix, gateway, and DNS settings match the network.
