# Day 3 — Network discovery and packet inspection

## Objectives
Day 3 extended the home-server lab into network visibility:
- Discover live devices on the authorized home LAN with Nmap.
- Record host and port snapshots for change review.
- Run scans hourly with cron.
- Inspect SSH traffic with tcpdump.

## Scope and privacy
The original lab used a private RFC1918 /24 home subnet. This public documentation replaces the subnet, addresses, account names, and wireless interface with placeholders. Only scan networks and devices you own or have permission to assess. Do not commit raw packet captures, real IP inventories, MAC addresses, or unredacted scan output.

## Work completed

### Nmap discovery
- Used Nmap 7.94SVN for local network discovery.
- Ran an ARP/ping discovery scan across the home /24 network.
- Identified five active devices during the recorded scan.
- Followed discovery with TCP checks against discovered devices, including the top 1,000 TCP ports.
- Confirmed SSH on TCP port 22 on the home server.
- Observed a device disappear from a later discovery result.

Discovery command:
~~~bash
sudo nmap -sn -PR -oX discovery.xml <authorized_lan_cidr>
~~~

Cron-compatible output redirection:
~~~bash
sudo /usr/bin/nmap -sn -PR -oX - "<authorized_lan_cidr>" > "$DISCOVERY"
~~~

Targeted top-1,000 TCP scan:
~~~bash
sudo nmap -sT -Pn --top-ports 1000 -sV <host_ip>
~~~

### Scheduled LAN monitoring
An hourly cron entry was configured:
~~~cron
0 * * * * /home/<server_user>/lan-monitor/lan-scan.sh >> /home/<server_user>/lan-monitor/cron.log 2>&1
~~~

The monitoring workspace used:
- baseline.tsv — initial reference snapshot.
- previous.tsv — previous host-presence snapshot.
- previous-ports.tsv — previous port snapshot.
- changes.log — recorded changes when a difference was detected.
- cron.log — scheduled execution output.

The scheduled command was observed executing. A genuine change detected specifically by a scheduled cron run was not confirmed; the observed device disappearance came from discovery testing.

### XML parsing issue
Nmap produced valid XML, but one extraction attempt returned zero hosts/port sections/open ports even though Nmap's summary reported a live host. The parser issue was not confirmed fixed during Day 3.

A parser failure must not be interpreted as every device disappearing. Validate XML before updating baselines, and retain the last known-good snapshot if parsing fails.

### tcpdump packet inspection
A focused capture filter was used to observe SSH traffic involving the client:
~~~bash
sudo tcpdump -i <server_wireless_interface> -nn 'host <client_ip> and tcp port 22'
~~~

- -i selects the server network interface.
- -nn disables hostname and service-name resolution.
- host limits capture to traffic involving the selected client.
- tcp port 22 narrows capture to SSH.

A short bounded capture can be made locally:
~~~bash
sudo timeout 30 tcpdump -i <server_wireless_interface> -nn -s 0 -w ssh-test.pcap 'host <client_ip> and tcp port 22'
sudo tcpdump -nn -r ssh-test.pcap
~~~

Packet captures can contain sensitive metadata. Keep them local; do not upload them to this public repository.

## Verification checklist
- [x] Nmap discovery completed on the authorized home LAN.
- [x] Five active devices observed in the recorded discovery.
- [x] TCP port/service checks performed, including the server's SSH service.
- [x] Hourly cron entry configured and execution log checked.
- [x] Baseline and previous-state files created.
- [x] tcpdump used with a host-and-SSH-port filter.
- [ ] Fix and retest XML host/port extraction before relying on automated comparisons.
- [ ] Capture a controlled host/port change during a scheduled run and verify end-to-end logging.

## Lessons learned
1. Discovery identifies reachable hosts; port scans identify responding services.
2. ARP discovery is useful on a directly connected IPv4 LAN.
3. Scheduled scans require reliable parsing and validation.
4. Never overwrite a valid baseline with malformed or incomplete results.
5. tcpdump reveals packet metadata, while SSH encrypts session contents.

## Next
Day 4 builds on these signals with a Python security-event monitor for Fail2ban, audit, SSH authentication, LAN changes, and firewall events.
