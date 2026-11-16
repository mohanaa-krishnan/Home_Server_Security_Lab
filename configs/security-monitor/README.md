# Security monitor deployment notes

## Live server file

The working Python monitor was created locally on the Ubuntu server at:

~~~text
/home/<server_user>/lan-monitor/security_monitor.py
~~~

The working notes were saved as `DAY_04.md` beside the script. The public repository documents the implementation and verified behaviour but does not include a fabricated replacement script: the exact private server source was not retrieved for this commit.

## Runtime requirements

- Python 3 (standard library only; no third-party Python package was required).
- The five configured log sources documented in [Day 4 event sources](../../docs/day-04/event-sources-and-alert-rules.md).
- Read access to the configured log files and the LAN monitor change log.

## Compile and run

~~~bash
python3 -m py_compile ~/lan-monitor/security_monitor.py
python3 ~/lan-monitor/security_monitor.py
~~~

The process was run manually in the foreground. It was not configured as a systemd service or other automatic startup job during Day 4.

## Important

Do not make `/var/log` world-readable to make the monitor work. Use a carefully reviewed least-privilege approach for log access. Do not commit live log files, alert output containing private details, or LAN inventory snapshots to this public repository.
