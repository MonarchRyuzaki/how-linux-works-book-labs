# Chapter 9: Understanding Your Network and Its Configuration

## Structural Outline & Core Concepts

- **9.1 Network Basics**: Introduces basic components like hosts, LAN, and the router that connects a LAN to an uplink (WAN).
- **9.2 Packets**: Describes how data is transmitted in small chunks called packets, containing headers and payload data.
- **9.3 Network Layers**: Explains the stack of layers (Application, Transport, Network, and Physical) that data moves through.
- **9.4 The Internet Layer**: Focuses on IPv4, IP addresses, subnets, and routing. 
  - **9.4.1 Viewing IP Addresses**: Shows how to use the `ip` command to view network interfaces.
  - **9.4.2 Subnets**: Explains how subnets are defined using a network prefix and a subnet mask.
  - **9.4.3 Common Subnet Masks and CIDR Notation**: Discusses CIDR format like /24 for representing subnet masks.
- **9.5 Routes and the Kernel Routing Table**: Explains how the Linux kernel uses a routing table to direct packets to local subnets or via routers.
- **9.6 The Default Gateway**: Describes the `0.0.0.0/0` default route for reaching destinations not explicitly in the routing table.
- **9.7 IPv6 Addresses and Networks**: Details the 128-bit IPv6 address structure and how it mitigates address exhaustion.
  - **9.7.1 Viewing IPv6 Configuration**: Shows IPv6 link-local and global unicast configurations.
  - **9.7.2 Configuring Dual-Stack Networks**: Explains configuring both IPv4 and IPv6 simultaneously.
- **9.8 Basic ICMP and DNS Tools**: Introduces tools for diagnosing connectivity and translating names to IPs.
  - **9.8.1 ping**: Explains using ICMP echo requests to test host reachability.
  - **9.8.2 DNS and host**: Discusses resolving hostnames to IP addresses using the `host` command.
- **9.9 The Physical Layer and Ethernet**: Explains MAC addresses, frames, and basic physical layer transmission.
- **9.10 Understanding Kernel Network Interfaces**: Focuses on the kernel mapping between internet configurations and physical devices (e.g., `enp0s31f6`).
- **9.11 Introduction to Network Interface Configuration**: Outlines the steps for linking hardware drivers, configuring physical setup, and assigning IPs.
- **9.12 Boot-Activated Network Configuration**: Describes historical and modern systems (like Netplan) to apply network configurations automatically on boot.
- **9.13 Problems with Manual configuration**: Explains the challenges dynamic/wireless setups introduce compared to static network configs.
- **9.14 Network Configuration Managers**: Focuses on `NetworkManager` for managing complex, dynamic desktop network settings.
- **9.15 Resolving Hostnames**: Explains how DNS resolution is prioritized and cached on the system.
  - **9.15.1 /etc/hosts**: Local overrides for hostname resolution.
  - **9.15.2 resolv.conf**: Legacy configuration for external DNS name servers.
  - **9.15.3 Caching and Zero-Configuration DNS**: Explains `systemd-resolved` and mDNS caching local lookups.
  - **9.15.4 /etc/nsswitch.conf**: Defines the resolution order (usually files before dns).
- **9.16 Localhost**: Describes the `lo` loopback interface for local application communication.
- **9.17 The Transport Layer: TCP, UDP, and Services**: Introduces TCP streaming vs UDP datagrams for application-level data transport.
  - **9.17.1 TCP Ports and Connections**: Details connection-oriented state, well-known vs ephemeral ports, and `/etc/services`.
  - **9.17.2 UDP**: Explains connectionless, fast message transmission without error correction (e.g., NTP).
- **9.18 Revisiting a Simple Local Network**: Recaps combining routing, NAT, and DHCP.
- **9.19 Understanding DHCP**: Explains Dynamic Host Configuration Protocol for dynamic IP leases.
- **9.20 Automatic IPv6 Network Configuration**: Explains stateless SLAAC configuration where hosts self-assign IPv6 addresses.
- **9.21 Configuring Linux as a Router**: Explains enabling IP forwarding (`net.ipv4.ip_forward=1`) to route traffic between subnets.
- **9.22 Private Networks (IPv4)**: Describes RFC 1918 non-routable private subnet spaces (10.x.x.x, 192.168.x.x).
- **9.23 Network Address Translation (IP Masquerading)**: Details using `iptables` and NAT to share one public IP address across a private network.
- **9.24 Routers and Linux**: Touches on embedded Linux distributions (OpenWRT) for router hardware.
- **9.25 Firewalls**: Covers filtering traffic at the kernel level for security.
  - **9.25.1 Linux Firewall Basics**: Introduces `iptables` chains (INPUT, OUTPUT, FORWARD) and policies.
  - **9.25.2 Setting Firewall Rules**: Demonstrates appending and inserting allow/deny rules based on ports and IPs.
  - **9.25.3 Firewall Strategies**: Emphasizes a default-deny policy (DROP) while explicitly allowing necessary traffic.
- **9.26 Ethernet, IP, ARP, and NDP**: Explains the Address Resolution Protocol mapping IP addresses to physical MAC addresses.
- **9.27 Wireless Ethernet**: Discusses specialized components of Wi-Fi (SSID, Authentication, Encryption).

## Commands & Utilities

- `ip address show`: View all active IP addresses and network interfaces.
- `ip route show`: View the system's current routing table.
- `ping`: Send ICMP echo requests to test host reachability.
- `host`: Perform DNS lookups translating hostnames to IP addresses.
- `netstat -nt` / `netstat -ntl`: View active and listening TCP connections.
- `sysctl -w net.ipv4.ip_forward=1`: Enable IP forwarding in the kernel to allow routing.
- `iptables -L`: List the active netfilter/firewall rules.
- `iptables -A INPUT -p tcp --destination-port 22 -j ACCEPT`: Append a rule to allow incoming SSH connections.
- `ip neigh`: View the ARP cache showing IP-to-MAC address mappings.
- `iw`: Manage wireless configurations (e.g., `iw dev scan` to discover SSIDs).
- `dhclient`: Request a DHCP lease manually.
