# The Transport Layer: TCP, UDP, Ports & Sockets (Section 9.17)

While the Internet Layer (IP addresses) gets packets from one host to another, the Transport Layer determines what happens to those packets once they arrive. It acts as the bridge between raw network data and user-space applications (like a web server or database).

## 1. TCP vs. UDP

### TCP (Transmission Control Protocol)
- **Connection-Oriented:** TCP establishes a dedicated "connection" between a client and a server (like a phone call).
- **Reliable:** It ensures packets arrive in order. If a packet gets lost, TCP automatically requests a retransmission.
- **Overhead:** Because it guarantees delivery and ordering, it requires more overhead (the three-way handshake).
- **Use Cases:** HTTP/HTTPS, SSH, Database connections (PostgreSQL/MySQL).

### UDP (User Datagram Protocol)
- **Connectionless:** UDP fires off packets without establishing a connection (like dropping a letter in the mail).
- **Unreliable (Best Effort):** If a packet is lost or arrives out of order, UDP does not care and will not fix it.
- **Extremely Fast:** It has very little overhead.
- **Use Cases:** Video streaming, VoIP, DNS queries, and NTP (Network Time Protocol).

## 2. Ports and Connections
A single Linux server can run multiple applications simultaneously. Ports are how the OS routes incoming traffic to the correct application.
- **IP Address:** Identifies the machine (The apartment building).
- **Port:** Identifies the specific application (The apartment number).

A full TCP connection is uniquely identified by a 4-tuple: `(Source IP, Source Port, Destination IP, Destination Port)`.

### Well-Known vs. Ephemeral Ports
- **Well-Known Ports (0 - 1023):** Reserved for system services (e.g., 80 for HTTP, 443 for HTTPS, 22 for SSH). On Linux, only the `root` user can bind an application to these ports. The file `/etc/services` acts as a dictionary translating these port numbers to names.
- **Ephemeral Ports (1024+):** When your laptop acts as a client and connects to a remote server, your OS temporarily grabs a random high-numbered port (e.g., 47626) to use as the Source Port for that specific connection.

## 3. Monitoring Ports: `netstat` vs. `ss`

While older documentation (and Section 9.17) heavily references `netstat`, **`netstat` is considered deprecated in modern SRE environments.**

The modern, vastly superior replacement is **`ss` (Socket Statistics)**. `ss` queries the Linux kernel directly via netlink sockets, making it exponentially faster than `netstat` (which reads slowly from the `/proc` directory).

### The Ultimate SRE Debugging Command: `ss -tlnp`
When debugging a server to see what applications are listening for traffic, use:
```bash
sudo ss -tlnp
```

**Breaking down the flags:**
- **`-t` (TCP):** Show only TCP sockets. (Use `-u` for UDP).
- **`-l` (Listening):** Show only sockets waiting for incoming connections (hides the clutter of active/established connections).
- **`-n` (Numeric):** *CRITICAL SRE FLAG.* Forces `ss` to print raw IP addresses and raw port numbers (e.g., `80`) instead of trying to resolve them to human-readable names (e.g., `http`) via DNS or `/etc/services`. This makes the command instantaneous.
- **`-p` (Process):** Shows the exact Process ID (PID) and application name holding the port. (Requires `sudo` to see processes you don't own).
