# Chapter 6: How User Space Starts (init & systemd)

## Structural Outline
1. Introduction to init
2. Identifying Your init
3. systemd
   - Units and Unit Types
   - Booting and Unit Dependency Graphs
   - systemd Configuration (Unit Files, Variables, Specifiers)
   - systemd Operation (Jobs)
   - systemd Process Tracking and Synchronization (Type=simple, forking, notify, etc.)
   - systemd Dependencies (Wants, Requires, Before, After)
   - The `[Install]` Section and Enabling Units
   - systemd On-Demand and Resource-Parallelized Startup
4. System V init (Legacy)
   - Runlevels
   - /etc/init.d/ Scripts

## Core Concepts

**1. The Role of init**
Once the kernel finishes booting and hardware initialization, it passes control to user space by starting the very first process, PID 1 (which is `init`). This process is responsible for bringing up all other services (networking, SSH, databases, login prompts). `systemd` is the modern standard for this.

**2. systemd Units & Types**
systemd uses "units" to manage resources. The main types are:
- **Service Units (`.service`)**: Manages daemons (e.g., `sshd.service`).
- **Target Units (`.target`)**: Groups other units together (like runlevels). E.g., `multi-user.target`.
- **Socket Units (`.socket`)**: Listens on a port and activates a service unit only when traffic arrives (on-demand).
- **Mount Units (`.mount`)**: Manages filesystem mounts.

**3. Dependency Management (The Topological Graph)**
systemd parses unit files to build a dependency graph. 
- `Requires`: Strict. If the dependency fails, this unit also fails.
- `Wants`: Weak. If the dependency fails, this unit still starts. (Preferred for robustness).
- `Before` / `After`: Dictates *ordering* (not whether a unit is required).

**4. Process Tracking & Cgroups**
Unlike older init systems that lost track of processes if they forked, systemd places every service into a Linux Control Group (`cgroup`). This guarantees that if you stop a service, systemd kills every single child process cleanly.
- `Type=simple`: Stays in the foreground.
- `Type=forking`: Expected to fork to the background (systemd tracks the PID).

**5. System V init (Legacy)**
The old way of doing things. Relied on sequential bash scripts in `/etc/init.d/` and runlevels (0-6). `systemd` replaces runlevels with targets (e.g., Runlevel 3 = `multi-user.target`, Runlevel 5 = `graphical.target`).

## Commands & Utilities

- **`systemctl list-units`**: Show all active units.
- **`systemctl status <service>`**: View the state, PID, memory, and recent logs of a service.
- **`systemctl start/stop/restart <service>`**: Manage running state.
- **`systemctl enable/disable <service>`**: Configure whether a service starts automatically on boot (creates/removes symlinks in `.wants` directories).
- **`systemctl daemon-reload`**: Forces systemd to re-read unit files from disk. (MUST run this after editing a `.service` file).
- **`journalctl -u <service>`**: View logs specifically for that systemd unit.
- **`systemd-cgls`**: View the cgroup tree for systemd.
