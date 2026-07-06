# Task 01: The Midnight Ghost Crash

## Scenario (Mode B: Production Outage)
It's 3:00 AM. The primary backend `payment.service` crashed silently. The monitoring system automatically restarted the server to recover. The server is back online, but the tech lead needs you to find out exactly *why* the service crashed before the reboot. The legacy `syslog` files in `/var/log` are completely empty because the system recently migrated entirely to `journald`. 

## Objective
Write the exact `journalctl` command(s) you would use to:
1. View only the logs from the *previous* boot cycle.
2. Filter the output to show *only* messages from `payment.service`.
3. Filter the output to show only high-severity errors (severity level 3 or higher).

## Solution
1. `sudo journalctl -r -b -1 -u payment.service -p 0..3`

**Correction:**
* **Grade: 10/10!** Perfect command construction.
* **Breakdown:** 
  * `-r` (Reverse order, newest first - great for finding crashes)
  * `-b -1` (Previous boot cycle)
  * `-u payment.service` (Unit filter)
  * `-p 0..3` (Priority 0 to 3, which is Emerg to Err).
Spot on!
