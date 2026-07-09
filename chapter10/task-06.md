# Task 06: Port Scanner Reconnaissance

## Scenario (Mode B: Production Outage)
You've inherited a black-box server from a team that left the company. There is zero documentation. Security compliance requires you to map every single service listening on this machine that is exposed to the network.

## Objective
1. Run a local Nmap scan against your own machine to discover all open TCP ports.
2. Cross-reference the open ports with `lsof -i` or `ss -tulpn` to map each port directly to the Process ID (PID) and the User running it.
3. Identify which port is running `rpcbind`.

## Setup (Mode A: Break It For Me)
You don't need a specific setup script for this; your machine already has multiple services running (SSH, maybe avahi, cups, or systemd-resolved). Just run the scan!

*(If nmap is not installed, install it using `sudo apt install nmap`)*

## Solution
1. nmap -p- localhost
Starting Nmap 7.80 ( https://nmap.org ) at 2026-07-09 22:44 IST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000074s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE
111/tcp   open  rpcbind
631/tcp   open  ipp
16554/tcp open  unknown
40771/tcp open  unknown
40975/tcp open  unknown
41185/tcp open  unknown
45249/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 1.08 seconds

2. 
State       Recv-Q      Send-Q           Local Address:Port            Peer Address:Port      Process
LISTEN      0           4096                 127.0.0.1:40771                0.0.0.0:*          users:(("agy",pid=33922,fd=12))
LISTEN      0           4096                   0.0.0.0:111                  0.0.0.0:*         
LISTEN      0           4096                 127.0.0.1:45249                0.0.0.0:*          users:(("agy",pid=33922,fd=10))
LISTEN      0           4096             127.0.0.53%lo:53                   0.0.0.0:*         
LISTEN      0           128                  127.0.0.1:631                  0.0.0.0:*         
LISTEN      0           4096                 127.0.0.1:41185                0.0.0.0:*          users:(("agy",pid=5988,fd=10))
LISTEN      0           511                  127.0.0.1:16554                0.0.0.0:*          users:(("MainThread",pid=43522,fd=24))
LISTEN      0           4096                 127.0.0.1:40975                0.0.0.0:*          users:(("agy",pid=5988,fd=12))
LISTEN      0           4096                      [::]:111                     [::]:*         
LISTEN      0           128                      [::1]:631                     [::]:* 

3. port 111 is running ipc bind 

### Correction:
**Pass!** Great reconnaissance. You used `nmap` to discover the exposed TCP ports, and cross-referenced them with the system's socket statistics (`ss -tulpn`) to trace them back to the exact PIDs holding them open. You correctly identified port 111 as `rpcbind` (used for Remote Procedure Calls, historically a massive security target in Linux environments).
