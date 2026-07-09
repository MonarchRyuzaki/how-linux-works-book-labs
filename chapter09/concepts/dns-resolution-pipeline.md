# The Linux DNS Resolution Pipeline (Section 9.15)

When an application (like `curl`, a web browser, or a backend microservice) needs to connect to a domain like `google.com` or `database.internal`, it must translate that human-readable name into a machine-readable IP address. 

In Linux, this is not a single simple step. It is a multi-step pipeline. Understanding this chain is critical for SREs troubleshooting "Cannot resolve hostname" errors.

## The Full Resolution Chain

If you run `curl google.com`, here is the exact path the DNS lookup takes:

### 1. `/etc/nsswitch.conf` (The Name Service Switch)
Before your machine even attempts to talk to a DNS server, it checks this configuration file to see *how* it should perform the lookup.
- You will typically see a line like: `hosts: files dns`
- This tells the Linux kernel: "When looking up a host, **first** check local files. If you don't find it there, **then** use DNS."

### 2. `/etc/hosts` (The Local Override File)
Because `/etc/nsswitch.conf` prioritizes `files`, the kernel first checks the `/etc/hosts` file.
- This file contains hardcoded IP-to-hostname mappings (like `127.0.0.1 localhost`).
- If `google.com` is hardcoded here, the kernel returns that IP immediately and stops.
- **SRE Tip:** You should keep this file as small as possible. Never use it to "boost performance", as forgotten entries here cause baffling outages later.

### 3. `/etc/resolv.conf` & The Local Cache (`127.0.0.53`)
If the hostname is not in `/etc/hosts`, the kernel moves to the `dns` step. It looks at `/etc/resolv.conf` to find the IP address of its designated "phonebook" (nameserver).
- On modern Linux, you will usually see `nameserver 127.0.0.53`. 
- This is **not** a public internet server. It is a loopback address pointing to a local background service running on your own machine called **`systemd-resolved`**.
- `systemd-resolved` acts as a smart, local caching proxy. It checks its local memory cache. If you visited the site recently, it returns the IP instantly with 0ms latency.

### 4. The Router (The DNS Forwarder)
If the local `systemd-resolved` cache misses, it must ask a real external nameserver. You can see which external servers it uses by running `resolvectl status`.
- On a home or office Wi-Fi, DHCP usually configures this to be your **Router's IP address** (e.g., `192.168.29.1`).
- The router is not a true global DNS server. Instead, it runs a lightweight proxy (like `dnsmasq`). It checks its own network-wide cache, and if it misses, it forwards the request to the internet.

### 5. The ISP DNS / Public Internet
Finally, the router asks your Internet Service Provider's massive public DNS servers (or a configured upstream like Google's `8.8.8.8` or AWS Route 53). 
- The ISP DNS finds the true IP address and hands it back down the chain.
- The Router caches it ➔ `systemd-resolved` caches it ➔ `curl` receives the IP and makes the HTTP connection.

## Summary Diagram
```
Application (curl) 
       ↓ 
/etc/nsswitch.conf (Rules: files -> dns)
       ↓ 
/etc/hosts (Local Overrides)
       ↓ 
/etc/resolv.conf (Points to 127.0.0.53)
       ↓ 
systemd-resolved (Local Cache Daemon)
       ↓ 
192.168.29.1 (Router / DNS Forwarder)
       ↓ 
ISP DNS Server (Public Internet)
```
