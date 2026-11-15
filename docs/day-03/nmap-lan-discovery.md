# Nmap LAN discovery and host monitoring

## Purpose
Use Nmap to identify devices visible on the home LAN, inspect exposed TCP services, and compare snapshots over time.

## Install and identify the authorized subnet
On Ubuntu:
~~~bash
sudo apt update
sudo apt install nmap
nmap --version
ip -br address
ip route
~~~

Determine the connected subnet from the server address and prefix. In this public example, <authorized_lan_cidr> represents a network you own or are authorized to scan.

## Host discovery
~~~bash
sudo nmap -sn -PR -oX discovery.xml <authorized_lan_cidr>
~~~
- -sn performs host discovery only.
- -PR uses ARP discovery on a local Ethernet network.
- -oX writes XML for later processing.

Cron-compatible form:
~~~bash
sudo /usr/bin/nmap -sn -PR -oX - "<authorized_lan_cidr>" > "$DISCOVERY"
~~~

The lab recorded five active devices during one scan.

## TCP service scan
~~~bash
sudo nmap -sT -Pn --top-ports 1000 -sV <host_ip>
~~~
- -sT performs a TCP connect scan.
- -Pn skips host discovery for a target already discovered.
- --top-ports 1000 checks the 1,000 most common TCP ports.
- -sV attempts service/version identification.

The lab identified SSH on TCP port 22 on the home server.

## Snapshot design
| File | Purpose |
|---|---|
| baseline.tsv | Initial reference state |
| previous.tsv | Most recent host-presence snapshot |
| previous-ports.tsv | Most recent port snapshot |
| changes.log | Differences recorded by the monitor |
| cron.log | Scheduled-run output |

Compare the current scan with the previous valid snapshot. Update state only after parsing and validation succeed.

## XML parser failure
One scan produced valid Nmap XML, but the extraction code reported zero hosts and zero port sections while Nmap's own summary reported a host up. The extraction bug was not confirmed fixed during Day 3.

Do not turn a parser error into an empty host list. Validate XML and its expected structure first; if validation fails, log the error and retain the previous known-good snapshot. This prevents false mass-disappearance alerts and baseline corruption.

## Hourly cron schedule
~~~cron
0 * * * * /home/<server_user>/lan-monitor/lan-scan.sh >> /home/<server_user>/lan-monitor/cron.log 2>&1
~~~
This runs at minute 0 of every hour. Replace <server_user> with the local service account when installing.

Inspect the crontab and log:
~~~bash
crontab -l
tail -n 100 /home/<server_user>/lan-monitor/cron.log
~~~

The hourly entry and execution were checked. A controlled change detected by a scheduled execution was not yet demonstrated.

## Operational safeguards
- Restrict scans to networks you own or are authorized to test.
- Use absolute paths in cron jobs.
- Log errors and exit codes.
- Never replace a valid baseline when Nmap fails or XML parsing is incomplete.
- Keep raw host inventories and MAC addresses out of a public repository.
