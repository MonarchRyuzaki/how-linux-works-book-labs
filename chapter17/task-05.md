# Task 05: The Rootless Escape

**Mode:** Production Outage Translation (Mode B)

## Scenario
Your security team mandates that all new containers must be run without the `dockerd` daemon, as the daemon requires superuser privileges and presents a massive attack surface. 

You decide to migrate a high-throughput microservice from Docker to **Podman** running in *rootless* mode.

The container starts perfectly, but during load testing, it is inexplicably slow and drops packets. 
You profile the network traffic and notice that instead of routing efficiently through a kernel-space virtual bridge, the traffic is being heavily bottlenecked by a user-space forwarding daemon called `slirp4netns`.

## Instructions
1. Explain *why* Podman in rootless mode is forced to use a TAP interface and a user-space daemon (`slirp4netns`) instead of the standard fast kernel virtual bridge (`docker0` / `veth`) used by Docker.
2. What is the fundamental security trade-off being made here between networking performance and privilege escalation risk?

---

## Your Solution
not sure about this. 

**Correction:**
This is an advanced networking concept, so let's break it down!

1. **Why user-space (`slirp4netns`)?** Standard Docker creates a virtual ethernet bridge (`docker0` / `veth` pairs) inside the Linux kernel. However, modifying the kernel's network stack requires root privileges (specifically `CAP_NET_ADMIN`). Since Podman is running in rootless mode, the OS completely blocks it from touching kernel networking. To get internet access into the container without root, Podman has to use a `TAP` interface and route the traffic through a user-space application (`slirp4netns`). 
2. **The Trade-off:** By keeping everything in user space, we completely eliminate the need for the `dockerd` daemon and root privileges, making the system immensely more secure against privilege escalation attacks. The cost is performance: routing packets through a user-space daemon requires constant, expensive context switching between user space and kernel space, heavily bottlenecking network throughput.
