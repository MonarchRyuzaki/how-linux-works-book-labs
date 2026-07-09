# Active Practice: The Bare-Metal AWS VPC (Mode E)

You requested a "DO THIS" lab to build an AWS VPC from scratch using bare-metal Linux. To accomplish this on your single machine, we will use **Network Namespaces (`ip netns`)**—the exact kernel technology Docker uses to build isolated networks.

This lab will build a miniature VPC entirely inside your laptop.

## Architecture Overview
- **`vpc-router` Namespace:** Acts as the VPC backbone (IGW + NAT).
- **`public-host` Namespace:** Acts as an EC2 instance in a Public Subnet (`10.0.1.0/24`).
- **`private-host` Namespace:** Acts as an EC2 instance in a Private Subnet (`10.0.2.0/24`).

---

## The Setup Script (Laying the Cables)
Run this bash script to create the namespaces and the virtual ethernet cables (`veth` pairs) connecting them. 

```bash
#!/bin/bash

# ==========================================
# 1. Create the isolated namespaces (Mini-VMs)
# ==========================================
# 'ip netns add' tells the kernel to carve out a brand new, empty networking stack. 
# These act like virtual machines that only have networking (no CPU/RAM overhead).
sudo ip netns add vpc-router    # Creates the router's isolated network
sudo ip netns add public-host   # Creates the public EC2 instance's isolated network
sudo ip netns add private-host  # Creates the private EC2 instance's isolated network

# ==========================================
# 2. Create the virtual cables (veth pairs)
# ==========================================
# A 'veth' (virtual ethernet) always comes in pairs. Think of it like a long ethernet cable 
# with a plug on both ends. What goes in one end comes out the other.
# We are making two cables: one for the public subnet, one for the private subnet.
sudo ip link add veth-pub-router type veth peer name veth-pub-host
sudo ip link add veth-priv-rtr type veth peer name veth-priv-host

# ==========================================
# 3. Plug the cables into the correct namespaces
# ==========================================
# By default, the cables we just created are sitting in our host machine's main network.
# We need to take the ends of the cables and plug them into our isolated namespaces.
# Public cable:
sudo ip link set veth-pub-router netns vpc-router   # Plug one end into the router
sudo ip link set veth-pub-host netns public-host    # Plug the other end into the public host
# Private cable:
sudo ip link set veth-priv-rtr netns vpc-router  # Plug one end into the router
sudo ip link set veth-priv-host netns private-host  # Plug the other end into the private host

# ==========================================
# 4. Power up the interfaces
# ==========================================
# When you plug a cable in, the interface is physically 'down' (turned off) by default.
# We use 'ip netns exec <namespace>' to enter the namespace and run 'ip link set <interface> up' to turn them on.

# Power up the Router's interfaces
sudo ip netns exec vpc-router ip link set lo up               # Turn on the loopback (127.0.0.1)
sudo ip netns exec vpc-router ip link set veth-pub-router up  # Turn on the public-facing port
sudo ip netns exec vpc-router ip link set veth-priv-rtr up # Turn on the private-facing port

# Power up the Public Host's interfaces
sudo ip netns exec public-host ip link set lo up
sudo ip netns exec public-host ip link set veth-pub-host up

# Power up the Private Host's interfaces
sudo ip netns exec private-host ip link set lo up
sudo ip netns exec private-host ip link set veth-priv-host up

echo "✅ VPC Physical Wiring Complete."
```

*(Note: To run a command inside a specific namespace, prefix it with `sudo ip netns exec <name>`)*

---

## Phase 1: Subnet Allocation (DHCP is disabled!)
In AWS, DHCP assigns IPs instantly. Here, you must do it manually.
**Your Mission:** Assign the following IP addresses to the correct interfaces using the `ip address add <ip/mask> dev <interface>` command.

**The Public Subnet (`10.0.1.0/24`)**
- Assign `10.0.1.1/24` to `veth-pub-router` (inside `vpc-router`).
- Assign `10.0.1.10/24` to `veth-pub-host` (inside `public-host`).

**The Private Subnet (`10.0.2.0/24`)**
- Assign `10.0.2.1/24` to `veth-priv-rtr` (inside `vpc-router`).
- Assign `10.0.2.10/24` to `veth-priv-host` (inside `private-host`).

*Verification:* Run `sudo ip netns exec public-host ping 10.0.1.1` to ensure it can reach the router.

Solution:
sudo ip netns exec public-host ping 10.0.1.1
PING 10.0.1.1 (10.0.1.1) 56(84) bytes of data.
64 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=0.040 ms

64 bytes from 10.0.1.1: icmp_seq=2 ttl=64 time=0.049 ms
64 bytes from 10.0.1.1: icmp_seq=3 ttl=64 time=0.080 ms
64 bytes from 10.0.1.1: icmp_seq=4 ttl=64 time=0.079 ms
^C
--- 10.0.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3066ms
rtt min/avg/max/mdev = 0.040/0.062/0.080/0.017 ms

 sudo ip netns exec private-host ping 10.0.2.1
PING 10.0.2.1 (10.0.2.1) 56(84) bytes of data.
64 bytes from 10.0.2.1: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 10.0.2.1: icmp_seq=2 ttl=64 time=0.081 ms
^C
--- 10.0.2.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.047/0.064/0.081/0.017 ms

sudo ip netns exec public-host ping 10.0.2.10
ping: connect: Network is unreachable

sudo ip netns exec private-host ping 10.0.1.10
ping: connect: Network is unreachable


---

## Phase 2: Route Tables & IP Forwarding
Right now, `private-host` cannot ping `public-host`. The routing tables are empty, and the router refuses to forward packets.

**Your Mission:**
1. Tell `public-host` that its default gateway is `10.0.1.1`.
2. Tell `private-host` that its default gateway is `10.0.2.1`.
   *(Hint: `ip route add default via <ip>`)*
3. Enable IP Forwarding inside the `vpc-router` namespace so it acts like a true AWS Router.
   *(Hint: `sysctl`)*

*Verification:* `sudo ip netns exec private-host ping 10.0.1.10`. The ping should now succeed!

Solution:
sudo ip netns exec public-host ip route add default via 10.0.1.1

sudo ip netns exec private-host ip route add default via 10.0.2.1

sudo ip netns exec vpc-router sysctl net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1

sudo ip netns exec private-host ping 10.0.1.10
PING 10.0.1.10 (10.0.1.10) 56(84) bytes of data.
64 bytes from 10.0.1.10: icmp_seq=1 ttl=63 time=0.030 ms
64 bytes from 10.0.1.10: icmp_seq=2 ttl=63 time=0.055 ms
^C
--- 10.0.1.10 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1033ms
rtt min/avg/max/mdev = 0.030/0.042/0.055/0.012 ms

sudo ip netns exec public-host ping 10.0.2.10
PING 10.0.2.10 (10.0.2.10) 56(84) bytes of data.
64 bytes from 10.0.2.10: icmp_seq=1 ttl=63 time=0.029 ms
64 bytes from 10.0.2.10: icmp_seq=2 ttl=63 time=0.062 ms
^C
--- 10.0.2.10 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1014ms
rtt min/avg/max/mdev = 0.029/0.045/0.062/0.016 ms


---

## Phase 3: The NAT Gateway
Currently, when `private-host` pings `public-host`, the `public-host` sees the ping coming directly from the private IP `10.0.2.10`. We want to hide the private subnet behind a NAT Gateway.

**Your Mission:**
1. Configure an `iptables` rule on the `vpc-router` to **MASQUERADE** all traffic leaving out of the `veth-pub-router` interface. 
2. Open a second terminal and run a packet sniffer on the public host:
   `sudo ip netns exec public-host tcpdump -i veth-pub-host icmp`
3. Ping from the private host to the public host. Look at the `tcpdump` output.
   
*Verification:* If NAT is working correctly, `tcpdump` will show the ping originating from the router (`10.0.1.1`), completely hiding the private host's true IP!

Solution: 
sudo ip netns exec vpc-router iptables -t nat -A POSTROUTING -o veth-pub-router -j MASQUERADE

sudo ip netns exec private-host ping 10.0.1.10
PING 10.0.1.10 (10.0.1.10) 56(84) bytes of data.
64 bytes from 10.0.1.10: icmp_seq=1 ttl=63 time=0.049 ms
64 bytes from 10.0.1.10: icmp_seq=2 ttl=63 time=0.081 ms
64 bytes from 10.0.1.10: icmp_seq=3 ttl=63 time=0.106 ms
64 bytes from 10.0.1.10: icmp_seq=4 ttl=63 time=0.089 ms
64 bytes from 10.0.1.10: icmp_seq=5 ttl=63 time=0.137 ms
^C
--- 10.0.1.10 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4125ms
rtt min/avg/max/mdev = 0.049/0.092/0.137/0.028 ms

19:01:17.119014 IP 10.0.1.1 > 10.0.1.10: ICMP echo request, id 13708, seq 1, length 64
19:01:17.119025 IP 10.0.1.10 > 10.0.1.1: ICMP echo reply, id 13708, seq 1, length 64
19:01:18.172206 IP 10.0.1.1 > 10.0.1.10: ICMP echo request, id 13708, seq 2, length 64
19:01:18.172222 IP 10.0.1.10 > 10.0.1.1: ICMP echo reply, id 13708, seq 2, length 64
19:01:19.196445 IP 10.0.1.1 > 10.0.1.10: ICMP echo request, id 13708, seq 3, length 64
19:01:19.196465 IP 10.0.1.10 > 10.0.1.1: ICMP echo reply, id 13708, seq 3, length 64
19:01:20.220410 IP 10.0.1.1 > 10.0.1.10: ICMP echo request, id 13708, seq 4, length 64
19:01:20.220431 IP 10.0.1.10 > 10.0.1.1: ICMP echo reply, id 13708, seq 4, length 64
19:01:21.244472 IP 10.0.1.1 > 10.0.1.10: ICMP echo request, id 13708, seq 5, length 64
19:01:21.244506 IP 10.0.1.10 > 10.0.1.1: ICMP echo reply, id 13708, seq 5, length 64

10 packets captured
10 packets received by filter
0 packets dropped by kernel

---

## Phase 4: Port Debugging & Security Groups (Firewalls)
Let's lock down the `public-host` like a secure EC2 instance.

**Your Mission:**
1. Start a fake API server on the public host: 
   `sudo ip netns exec public-host python3 -m http.server 80 &`
2. Run the ultimate SRE command (`ss -tlnp`) inside `public-host` to verify the port is listening.
3. Apply a **Default Deny (DROP)** policy to the `INPUT` chain of `public-host`.
4. Try to `curl 10.0.1.10` from the `private-host`. It should hang/timeout.
5. Write the precise `iptables` rule to explicitly allow TCP port 80 into the `public-host`.

*Verification:* The `curl` from the private host should instantly succeed once the firewall hole is poked.

sudo ip netns exec public-host python3 -m http.server 80 &
[1] 28232

Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80) ...
sudo ip netns exec public-host ss -tlnp
State     Recv-Q    Send-Q         Local Address:Port         Peer Address:Port    Process
LISTEN    0         5                    0.0.0.0:80                0.0.0.0:*        users:(("python3",pid=28253,fd=3))

sudo ip netns exec public-host iptables -P INPUT DROP

sudo ip netns exec private-host curl 10.0.1.10:80

sudo ip netns exec public-host iptables -A INPUT -p tcp --dport 80 -j ACCEPT

sudo ip netns exec private-host curl 10.0.1.10:80
10.0.1.1 - - [09/Jul/2026 19:15:22] "GET / HTTP/1.1" 200 -
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href=".git/">.git/</a></li>
<li><a href=".gitignore">.gitignore</a></li>
<li><a href="chapter02/">chapter02/</a></li>
<li><a href="chapter03/">chapter03/</a></li>
<li><a href="chapter04/">chapter04/</a></li>
<li><a href="chapter05/">chapter05/</a></li>
<li><a href="chapter06/">chapter06/</a></li>
<li><a href="chapter07/">chapter07/</a></li>
<li><a href="chapter08/">chapter08/</a></li>
<li><a href="chapter09/">chapter09/</a></li>
<li><a href="How-Linux-Works-What-Every-Superuser-Should-Know.pdf">How-Linux-Works-What-Every-Superuser-Should-Know.pdf</a></li>
<li><a href="log.md">log.md</a></li>
<li><a href="PROMPT.md">PROMPT.md</a></li>
<li><a href="README.md">README.md</a></li>
<li><a href="WORKFLOW.md">WORKFLOW.md</a></li>
</ul>
<hr>
</body>
</html>


---

## Cleanup Script (Destroy VPC)
When you are done playing AWS God, tear it all down:
```bash
sudo ip netns delete vpc-router
sudo ip netns delete public-host
sudo ip netns delete private-host
```

**Corrections:**
Excellent execution! You successfully built a complete AWS-style VPC from raw Linux primitives.

**The "Aha!" Moment:**
Take a look at your output in Phase 4 when you curled the python server from the `private-host`. The Python web server logged the incoming connection as coming from `10.0.1.1` (the VPC router's public interface). It did *not* see `10.0.2.10` (the private host's true IP). 

That is the ultimate proof that your `iptables -j MASQUERADE` rule from Phase 3 was still working flawlessly! The private subnet was completely hidden behind the router's NAT. You essentially built a working NAT Gateway and an AWS Security Group using nothing but `ip netns`, `sysctl`, and `iptables`. Incredible work!
