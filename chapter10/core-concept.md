# Chapter 10: Network Applications and Services

## Structural Outline
1. **10.1 The Basics of Services:** Connecting to TCP services directly (e.g., via `telnet`).
2. **10.2 A Closer Look:** Tracing application layer vs transport layer data (e.g., via `curl`).
3. **10.3 Network Servers:** Worker process models (forking vs socket units) and in-depth SSH configuration (`sshd`, host keys, `fail2ban`).
4. **10.4 Pre-systemd Network Connection Servers:** Legacy super-servers like `inetd`, `xinetd`, and TCP wrappers (`tcpd`).
5. **10.5 Diagnostic Tools:** Using `lsof`, `tcpdump`, `netcat`, and `nmap` for application-layer debugging.
6. **10.6 Remote Procedure Calls:** Understanding `rpcbind` and port-mapping.
7. **10.7 Network Security:** Fundamental security rules, intrusion types, and common vulnerabilities.
8. **10.9 Network Sockets:** The bridge between kernel transport and user-space applications (Stream vs Datagram sockets).
9. **10.10 Unix Domain Sockets:** Lightning-fast local Interprocess Communication (IPC) that bypasses the network stack entirely.

## Core Concepts
- **Application Layer Independence:** The application layer (HTTP, SSH) doesn't care about the underlying packets. The kernel handles TCP/IP; the application only handles the data stream payload.
- **Server Models:** High-performance servers either pre-fork worker processes (like Apache) or use modern event-driven asynchronous models (like Nginx). Legacy services often used `inetd` to handle connections and pass them to sleeping daemons.
- **Secure Shell (SSH) Internals:** SSH relies on asymmetric public-key cryptography for host and user authentication. The `known_hosts` file prevents Man-in-the-Middle attacks.
- **Network vs Unix Domain Sockets:** A network socket binds an IP and Port to communicate across networks. A Unix Domain Socket binds to a local filesystem path (like `/var/run/mysqld.sock`) allowing extremely fast IPC between local processes without traversing the kernel's TCP/IP stack.

## Commands & Utilities
- `telnet <host> <port>`: Originally for remote login, now used as a raw debugging client to manually talk TCP protocols (like HTTP).
- `curl --trace-ascii <file>`: Makes an HTTP request and logs the exact headers sent and received, highlighting the boundary between transport and application data.
- `sshd` / `sshd_config`: The SSH daemon and its configuration (port, root login, X11 forwarding).
- `ssh-keygen`: Generates RSA/ED25519 public/private keypairs.
- `fail2ban`: A script that monitors auth logs for brute-force attacks and dynamically bans the IPs using `iptables`.
- `lsof -i`: Lists all processes currently listening on or connected to network ports.
- `lsof -U`: Lists all active Unix domain sockets (local IPC).
- `tcpdump`: A packet sniffer that puts the NIC in promiscuous mode. Uses primitives (e.g., `tcpdump udp port 53`).
- `nc` (netcat): The TCP/UDP Swiss Army knife. Can listen for connections (`-l`), act as a client, or stream files.
- `nmap`: A security scanner used to discover hosts and open ports on a network.
- `rpcinfo -p`: Lists registered RPC services and their mapped ports.

## Deep Dive: Unix Domain Sockets (IPC)
A **Unix Domain Socket** is a file-based socket (`.sock`) used for lightning-fast Interprocess Communication (IPC) between applications running on the exact same server.

**Why SREs Prefer Unix Domain Sockets Over TCP/IP (localhost):**
1. **Massive Performance:** Bypasses the entire TCP/IP stack (no IP headers, no TCP checksums, no iptables routing). The kernel acts as a direct memory pipe between the two processes.
2. **Security via File Permissions:** Instead of an open network port (`8080`), the socket is a physical file (e.g., `/var/run/docker.sock`). You can secure it using standard Linux permissions (`chmod`, `chown`).
3. **Invisible to Scanners:** They do not use ports, meaning they are completely invisible to `netstat -tlnp` and `nmap`.

**How to Use and Troubleshoot Them:**
- **Finding active sockets:** You must use `lsof -U` (List Open Files - Unix) or `ss -x`.
- **Connecting as a client (HTTP):** `curl --unix-socket /var/run/docker.sock http://localhost/version`
- **Connecting as a raw client:** `nc -U /tmp/my_app.sock`
- **Creating a raw listening server:** `nc -lU /tmp/my_app.sock`
- **The SRE Config Fix:** If an app is hardcoded to look for a socket at `/tmp/db.sock` but the DB created it at `/var/run/db.sock`, use a symbolic link to route it: `ln -s /var/run/db.sock /tmp/db.sock`
