# Chapter 17: Virtualization

## Structural Outline & Core Concepts

### 17.1 Virtual Machines
Virtual machines (VMs) provide isolated environments by simulating entire machines, including a processor, memory, and I/O interfaces, on which a complete operating system and kernel run. These are often referred to as system virtual machines.

#### 17.1.1 Hypervisors
A hypervisor (or Virtual Machine Monitor) oversees virtual machines. Type 1 hypervisors act like an OS built specifically to run VMs efficiently (used in cloud computing like AWS), while Type 2 hypervisors run on top of a normal OS like Linux (e.g., VirtualBox). 

#### 17.1.2 Hardware in a Virtual Machine
To improve performance, VMs often use paravirtualization, bypassing virtual hardware to access host resources more directly (e.g., networking or block devices). Modern CPUs provide hardware features like VT-x or AMD-V to eliminate the need for hypervisors to trap and emulate kernel-mode instructions natively, improving VM performance.

#### 17.1.3 Common Uses of Virtual Machines
VMs are commonly used for safe testing and trials without affecting production systems. They also allow running applications designed for different operating systems and form the foundational infrastructure for all cloud computing and server deployments.

#### 17.1.4 Drawbacks of Virtual Machines
VMs suffer from heavy overhead because they require booting, maintaining, and updating a full underlying Linux system and kernel. Isolating services on separate VMs can also be resource-wasteful and costly compared to lighter alternatives.

### 17.2 Containers
Containers offer operating system-level virtualization, creating restricted runtime environments using the host's existing kernel. They isolate processes using kernel features like namespaces and cgroups so processes believe they have their own independent system.

#### 17.2.1 Docker, Podman, and Privileges
Docker traditionally uses the `dockerd` daemon requiring superuser privileges to access kernel isolation features. Podman is a compatible alternative that can run in rootless mode, utilizing different user-space isolation techniques (like `slirp4netns` for networking) without requiring a background daemon.

#### 17.2.2 A Docker Example
Docker builds containers from images using instruction files called `Dockerfile`. It heavily utilizes overlay filesystems, stacking read-only image layers and placing a writable layer on top to capture runtime changes. Networking is typically handled by creating a bridge interface (`docker0`) and a NAT, using network namespaces to connect the host and the container.

#### 17.2.3 LXC
LXC is an older container package built around a C API and manual setup rather than Docker's automated image building. It aims to create environments that closely resemble complete Linux systems (including `init`) and provides flexible adaptation to different infrastructure needs.

#### 17.2.4 Kubernetes
Kubernetes is a system used to manage, orchestrate, and deploy large clusters of containers across multiple machines. It handles restarting failed containers, networking configurations, and load-balancing traffic, making it indispensable for large-scale web services.

#### 17.2.5 Pitfalls of Containers
While lighter than VMs, containers can still experience storage bloat if applications and their supporting libraries are large. Furthermore, runtime data (like databases or logs) must be handled carefully outside the ephemeral overlay filesystem, and misconfigured containers can still crash underlying host systems.

### 17.3 Runtime-Based Virtualization
This kind of virtualization isolates programming language environments and dependencies rather than whole systems or processes. For example, Python's `venv` modifies environment variables (like `VIRTUAL_ENV`) and uses symbolic links to ensure an application uses its own specific set of packages without conflicting with the system-wide libraries.

---

## Commands & Utilities
- `VBoxManage`: Command-line tool to automate VirtualBox VMs.
- `docker build -t <tag> .`: Reads a `Dockerfile` in the current directory and builds a container image, tagging it with the specified name.
- `docker run -it <image>`: Starts a container interactively (`-i`) and connects a terminal (`-t`). Add `--rm` to remove the container upon exit, or `--init` to run an init process as PID 1 to reap zombie children.
- `docker ps`: Lists currently running containers. Add `-a` to show exited ones.
- `docker images`: Lists all container images stored locally.
- `docker rm` / `docker rmi`: Removes an exited container / removes a downloaded image to free up space.
- `docker export`: Extracts the filesystem of a container.
- `podman`: A rootless, daemonless alternative command-line compatible with Docker.
- `chroot`: A tool to change the root directory for a process, restricting its filesystem access to a specific path (a "chroot jail").
- `python3 -m venv <dir>`: Creates a Python virtual environment. Activated by sourcing `<dir>/bin/activate`.

---

## Deep Dive: Docker Internals & SRE Troubleshooting

### 1. The PID Namespace & The Zombie Orphan Trap
Docker uses **PID Namespaces** to isolate processes. This means the first process launched inside the container becomes **PID 1** (its own isolated init system). 
- **The Problem:** In Linux, PID 1 is expected to act as the OS `init` system, specifically handling the `wait()` syscall to reap dead child processes (zombies). Most applications (like Node.js or Python web servers) are not programmed to do this. If they spawn workers that die, the zombies pile up infinitely because the container's PID 1 ignores them, eventually exhausting the host's PID limits.
- **The Fix:** Running `docker run --init` injects a tiny, proper init system (like `tini`) as PID 1 inside the container, pushing your application to PID 2 and safely reaping all zombies.

### 2. OverlayFS and Storage Bloat
Docker uses an **Overlay Filesystem**. It takes a read-only image layer (e.g., Alpine Linux) and places a writable layer ("upperdir") on top. When a container writes data, it goes into this upperdir.
- **The Problem:** When a container stops or crashes, Docker *does not* delete this writable layer (in case you want to restart it). If you have a system with high container churn (frequent deployments, cron jobs, or crash loops), you will accumulate thousands of exited containers, filling `/var/lib/docker/overlay2/` and causing disk exhaustion.
- **The Fix:** Regularly run `docker system prune -a` or `docker container prune` to permanently delete stopped containers and dangling images.

### 3. Network Namespaces & iptables NAT
Containers live in isolated **Network Namespaces**, meaning their `eth0` interface and private IP (e.g., `172.17.0.2`) are completely invisible to the physical host machine's `eth0`. 
- **The Mechanism:** To bridge this gap, Docker relies heavily on the kernel's `iptables` NAT (Network Address Translation). When you publish a port using `-p 8080:80`, Docker creates a DNAT rule in iptables that intercepts traffic hitting the host's physical port 8080 and transparently forwards it to the container's private IP on port 80. Without `-p`, the host OS simply rejects external traffic.

### 4. Rootless Containers & slirp4netns (The Security/Performance Trade-off)
Running the `dockerd` daemon requires massive superuser privileges, creating a security risk (e.g., mounting the Docker socket `/var/run/docker.sock` allows trivial root escape to the host). Tools like **Podman** allow rootless container execution.
- **The Trade-off:** Modifying the kernel's network stack to create fast virtual bridges (`veth` pairs) requires root privileges (`CAP_NET_ADMIN`). Because Podman is rootless, the OS blocks it from doing this. Instead, it must create a `TAP` interface and route traffic through a user-space daemon called `slirp4netns`. This entirely eliminates the root privilege risk, but the constant context-switching between user-space and kernel-space heavily bottlenecks network performance.
