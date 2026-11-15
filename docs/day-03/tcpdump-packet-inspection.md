# tcpdump packet inspection

## Purpose
tcpdump captures and displays packets visible to a network interface. In Day 3, it was used to inspect SSH traffic between the server and a client on the home LAN.

## Identify the interface
On Ubuntu:
~~~bash
ip -br link
ip -br address
~~~
Use the interface connected to the lab network. This public repository uses <server_wireless_interface> instead of the real interface name.

## Focused SSH capture
~~~bash
sudo tcpdump -i <server_wireless_interface> -nn 'host <client_ip> and tcp port 22'
~~~

Filter breakdown:
| Option/filter | Meaning |
|---|---|
| -i | Select the capture interface |
| -nn | Do not resolve addresses or ports to names |
| host <client_ip> | Match packets to or from the selected client |
| tcp port 22 | Match TCP traffic using SSH's default port |

## Capture a short sample
Only capture traffic you are authorized to inspect. A time-bounded example:
~~~bash
sudo timeout 30 tcpdump -i <server_wireless_interface> -nn -s 0 -w ssh-test.pcap 'host <client_ip> and tcp port 22'
sudo tcpdump -nn -r ssh-test.pcap
~~~

Other useful filters:
~~~bash
# Show packet timestamps and details
sudo tcpdump -i <server_wireless_interface> -nn -tttt 'tcp port 22'

# Limit the number of packets
sudo tcpdump -i <server_wireless_interface> -nn -c 25 'tcp port 22'

# Inspect DNS traffic
sudo tcpdump -i <server_wireless_interface> -nn 'udp port 53'
~~~

## What can be seen
A capture can reveal metadata such as source/destination IP addresses, ports, packet timing, TCP flags, and packet sizes. SSH encrypts session payloads, so a passive capture does not normally reveal commands or data exchanged inside a properly configured SSH session.

## Privacy and handling
PCAP files may reveal internal addresses, device activity, DNS queries, and other sensitive metadata. Keep captures in a private local directory, restrict permissions, and do not commit them to this public Git repository.

~~~bash
chmod 600 ssh-test.pcap
~~~

Secure deletion is not guaranteed on SSDs, copy-on-write filesystems, or storage with snapshots. Prefer not creating or uploading unnecessary captures.
