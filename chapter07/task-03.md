# Task 03: The SSL Certificate Paradox

## Scenario (Mode B: Production Outage)
Your internal API service `auth-api` suddenly starts refusing all connections from clients. The clients report the error: `SSL certificate problem: certificate is not yet valid`. You check the certificate on the server, and it was successfully issued and signed yesterday. It is definitely valid. 

You run `timedatectl` and notice the "System clock synchronized" field says `no`, and the system time is completely wrong (it thinks it is currently the year 2019). 

## Objective
1. Explain exactly why a desynchronized system clock causes SSL/TLS handshakes to fail with a "not yet valid" or "expired" error.
2. What command would you use via `timedatectl` to enable the Network Time Protocol (NTP) synchronization to fix this automatically?

## Solution
1. It is likely that system clock is not in the range of the SSL valid date bounds (like after date X and before date y), this is the reason why it is showing, not valid yet or expired error
2. `sudo timedatectl set-ntp true`, if the correct date is still not working with tls certificate, it might be that ssl/tls date range is wrong and need to be renewed correctly. 

**Correction:**
* **Grade: 10/10!**
* **1:** Exactly. Every SSL cert has a `Not Before` and `Not After` timestamp. If the server thinks it's 2019, but the cert was issued in 2026, the server's TLS library rejects it as "not yet valid".
* **2:** Spot on with `sudo timedatectl set-ntp true`. This activates `systemd-timesyncd` (or ntpd) to poll network time servers and aggressively slew the clock back to the present day.
