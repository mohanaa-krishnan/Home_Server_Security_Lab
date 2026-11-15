# LAN monitor configuration notes

This directory contains a sanitized cron example for the Day 3 LAN discovery monitor.

The live monitor was created on the Ubuntu server under:
~~~text
/home/<server_user>/lan-monitor/lan-scan.sh
~~~

Its runtime state was stored locally in baseline.tsv, previous.tsv, previous-ports.tsv, changes.log, and cron.log. Those files are intentionally not copied into this public repository because they can disclose internal network inventory.

See lan-scan.cron.example for the hourly schedule. Replace <server_user> and verify the script path and Nmap path before installing. This is a schedule example, not a copy of the live script.

Known issue: Nmap XML was valid but one parser/extraction attempt returned zero hosts/port sections despite Nmap reporting a live host. Keep the previous known-good snapshot when parsing fails; fix and test this before relying on automated change detection.
