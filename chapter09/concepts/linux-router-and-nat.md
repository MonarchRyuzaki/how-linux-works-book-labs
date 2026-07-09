# Linux as a Router & NAT (Sections 9.21 - 9.24)

## 1. IP Forwarding (Linux as a Router)

By default, a Linux machine will only accept network packets destined for its own IP address. However, by enabling a specific kernel parameter, you can turn any Linux machine into a fully functional router that bridges multiple subnets.

To enable routing (IP Forwarding) in the kernel:
```bash
sysctl -w net.ipv4.ip_forward=1
```

### The Subnet Bridging Example
Imagine a Linux server with two network interfaces:
- `enp0s31f6` connected to Subnet A (`10.23.2.0/24`) with IP `10.23.2.1`
- `enp0s1` connected to Subnet B (`192.168.45.0/24`) with IP `192.168.45.1`

If all the hosts on Subnet A use `10.23.2.1` as their Default Gateway, and Host A (`10.23.2.4`) wants to send a packet to Host E (`192.168.45.61`) on Subnet B:
1. Host A sends the packet to the router (`10.23.2.1`).
2. The Linux router receives the packet on `enp0s31f6`.
3. Because IP Forwarding is enabled, the kernel looks at its routing table, sees that the destination is on Subnet B, and pushes the packet back out through the `enp0s1` interface directly to Host E.

## 2. The Problem with Private Subnets (RFC 1918)

The routing mechanism above works perfectly for jumping between internal LANs. However, a major problem arises when Host A wants to talk to the public internet (e.g., Google). 

IP addresses like `10.23.2.4` and `192.168.45.61` are defined as **Private IP Addresses** under RFC 1918 (the 10.x.x.x, 172.16.x.x, and 192.168.x.x ranges).
- These IPs are completely invisible to the public internet. 
- Public internet routers are strictly programmed to drop any packet they see coming from or going to a Private IP.

If the Linux router simply forwarded Host A's packet out to the internet, Google's servers wouldn't know how to reply because they cannot route traffic back to a `10.x.x.x` address.

## 3. The Solution: NAT (Network Address Translation)

To allow Private Subnets to reach the internet, the Linux router must perform **NAT** (specifically, **IP Masquerading** using `iptables`). 

When Host A sends a packet to Google:
1. The router intercepts the packet before it leaves the public-facing interface.
2. The router strips off Host A's private IP (`10.23.2.4`) and replaces it with its own Public, internet-routable IP address.
3. The router keeps a ledger tracking this swap (using transport layer TCP/UDP ports to distinguish connections).
4. When Google replies to the router's Public IP, the router checks its ledger, swaps the destination IP back to `10.23.2.4`, and forwards the packet to Host A.

---

## 4. Mapping Linux Routing to AWS Concepts

AWS VPCs are essentially massive, software-defined networks powered by invisible Linux servers running `iptables` and `sysctl net.ipv4.ip_forward=1`. We can map a bare-metal Linux router directly to AWS terminology:

Imagine a Linux machine with three cables:
- **Cable 1:** Private Subnet A (`10.23.2.x`)
- **Cable 2:** Private Subnet B (`192.168.45.x`)
- **Cable 3:** Internet Uplink (connected to the ISP)

### The AWS "Public Subnet" and "Internet Gateway (IGW)"
If you open a browser *on the Linux router itself* and connect to the internet, the packets originate from the router and go straight out of Cable 3.
- **In AWS terms:** The internal processes of the router act like an EC2 instance in a **Public Subnet**. It has a direct route to the internet, and Cable 3 acts as the **Internet Gateway (IGW)**.

### The AWS "NAT Gateway"
When a device connected to Cable 1 (a private network) wants to reach the internet, it sends the packet to the Linux router. The router intercepts it, applies the `iptables -j MASQUERADE` rule to strip the private IP, and sends it out Cable 3.
- **In AWS terms:** The device on Cable 1 is an EC2 instance in a **Private Subnet**. The Linux router is intercepting the traffic and acting exactly as the **NAT Gateway**.

### `iptables` as the AWS Control Plane
When you click "Create NAT Gateway" or edit Route Tables in the AWS Console, AWS is running automation scripts in the background that configure `iptables` and routing tables on invisible hypervisors. On a bare-metal Linux server, *you* are the control plane writing the `iptables` rules.
