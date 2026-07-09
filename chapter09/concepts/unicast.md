# Unicast vs. Broadcast & Global Unicast

## 1. What is Unicast?
**Unicast simply means "one-to-one" communication.** 
When a host sends a packet specifically to one destination IP (like `8.8.8.8`), that is a Unicast transmission. It has one specific sender and one specific destination. Almost all normal internet traffic—loading a webpage, querying a database, SSHing into a server—is Unicast.

By contrast, **Broadcast** is "one-to-all" communication. If a packet is sent to a broadcast IP (like `192.168.1.255`), every single machine on that subnet receives and processes it.

*(For completeness, **Multicast** is "one-to-many." You subscribe to a specific channel, and only hosts that opted in receive the traffic, like a radio broadcast.)*

## 2. What is a "Global" Unicast Address?
This term is primarily used when discussing IPv6. In IPv6, a single network interface typically receives several IP addresses based on their "scope" (how far the packet is allowed to travel).

* **Global Unicast:** This is a public, internet-routable IP address. If a packet has a Global Unicast destination, it can be routed across the entire public internet to reach that exact machine anywhere in the world. (In AWS IPv4 terms, this is equivalent to a public Elastic IP attached to an EC2 instance or NAT Gateway).
* **Link-Local Unicast:** This is a Unicast (one-to-one) address, but it is *strictly restricted to your local physical network*. Routers will absolutely refuse to forward a Link-Local packet to the outside internet. In IPv6, these always start with `fe80::`. In AWS IPv4 terms, this is conceptually similar to an internal `10.0.x.x` private VPC IP that cannot leave the VPC without a NAT.

**Summary:** 
- **Unicast** = A private, one-to-one conversation between two computers.
- **Global Unicast** = A one-to-one conversation where the IP address is globally recognized and routable across the entire public internet.
