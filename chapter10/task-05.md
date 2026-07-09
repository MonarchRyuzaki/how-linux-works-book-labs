# Task 05: The Broken Database Socket (Unix Domain Sockets)

## Scenario (Mode B: Production Outage)
A high-performance monolithic application is deployed on the same server as its database to minimize latency. Instead of using TCP/IP (which has overhead), it connects to the database via Unix Domain Sockets (IPC). 

Suddenly, the app crashes with: `ERROR: Cannot connect to local socket /tmp/db.sock`.
You check `netstat`, but you can't find it because network tools often ignore IPC sockets.

## Objective
1. Understand how to list active Unix Domain Sockets.
2. Find out what the actual file path of the database's IPC socket is on this system.
3. Fix the configuration (or use a symbolic link) so the app can find the socket at `/tmp/db.sock`.

## Setup (Mode A: Break It For Me)
Run this command to simulate the database creating a Unix Domain Socket in an unexpected location:
```bash
# Simulates a DB listening on a Unix socket
nohup nc -lU /tmp/production_database.sock >/dev/null 2>&1 &
echo "Database listening on /tmp/production_database.sock"
# The app expects it to be at /tmp/db.sock!
```

## Solution
1. Using ss -xl for listening, a for all sockets
2. ss -axlp | grep databaseu_str LISTEN 0      5                          /tmp/production_database.sock 285579            * 0    users:(("nc",pid=69536,fd=3))
3. ln -s /tmp/production_database.sock /tmp/db.sock

### Correction:
**Pass!** Nailed it. You successfully used `ss -x` (Unix domain sockets) to find the rogue database socket hiding from standard network tools. Using the symbolic link (`ln -s`) to route the hardcoded application to the correct socket is an elite SRE trick to fix production outages without having to redeploy the app's code!
