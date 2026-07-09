# Chapter 8: A Closer Look at Processes and Resource Utilization

## Structural Outline
- 8.1 Tracking Processes
- 8.2 Finding Open Files with lsof
  - 8.2.1 Reading the lsof Output
  - 8.2.2 Using lsof
- 8.3 Tracing Program Execution and System Calls
  - 8.3.1 strace
  - 8.3.2 ltrace
- 8.4 Threads
  - 8.4.1 Single-Threaded and Multithreaded Processes
  - 8.4.2 Viewing Threads
- 8.5 Introduction to Resource Monitoring
  - 8.5.1 Measuring CPU Time
  - 8.5.2 Adjusting Process Priorities
  - 8.5.3 Measuring CPU Performance with Load Averages
  - 8.5.4 Monitoring Memory Status
  - 8.5.5 Monitoring CPU and Memory Performance with vmstat
  - 8.5.6 I/O Monitoring
  - 8.5.7 Per-Process Monitoring with pidstat
- 8.6 Control Groups (cgroups)
  - 8.6.1 Differentiating Between cgroup Versions
  - 8.6.2 Viewing cgroups
  - 8.6.3 Manipulating and Creating cgroups
  - 8.6.4 Viewing Resource Utilization
- 8.7 Further Topics

## Core Concepts
- **Tracking Processes:** `top` provides an interactive, real-time interface to view current processes and their CPU/memory usage statistics. 
- **Finding Open Files with lsof:** `lsof` lists open files (including network sockets, pipes, and libraries) and the processes holding them, which is vital for troubleshooting locked files or finding dependencies.
- **Tracing Program Execution and System Calls:** `strace` tracks kernel-level system calls (like `open`, `read`, `fork`) made by a process. `ltrace` tracks user-level shared library calls, helping diagnose why a program crashes or fails to access resources.
- **Threads:** Processes can be divided into threads that share the same resources and memory. The kernel schedules threads similarly to processes, and tools like `ps m` or `top -H` can reveal thread-level activity.
- **Introduction to Resource Monitoring:** Tools like `time` show elapsed (real), user, and system time for a process. Process priority (determined by the kernel) can be influenced using "nice" values (`NI`); lowering the nice value makes a process less polite (runs with higher priority). CPU performance is broadly measured by load average (`uptime`), which counts processes ready to run.
- **Monitoring Memory Status:** The kernel manages memory in pages using on-demand paging. "Minor" page faults occur when a page is in memory but unmapped, while "major" page faults occur when the kernel must read the page from disk (causing slow I/O and potential thrashing). `vmstat` helps monitor swapping and major I/O bottlenecks.
- **I/O and Per-Process Monitoring:** Tools like `iostat` break down disk reads and writes across block devices. `iotop` provides a top-like interface for I/O per thread/process, showing I/O priorities. `pidstat` gives granular, historical snapshots of a specific process's CPU, memory, or disk usage over time.
- **Control Groups (cgroups):** `cgroups` (specifically v2) are a kernel feature (mounted in `/sys/fs/cgroup`) to isolate and limit resource consumption for groups of processes. 
  - **Configuring Limits:** You interact with cgroups by writing values into specific files within the cgroup's directory. For example, to set a hard memory limit, run `echo 500M > memory.max`. To limit the number of PIDs, run `echo 3000 > pids.max`. To restrict CPU usage, write quotas into `cpu.max`. To add a process to the cgroup, write its PID into the `cgroup.procs` file (`echo <pid> > cgroup.procs`).

## Commands & Utilities
- `top`: Real-time interactive process viewer. Sort by CPU (`P`), Memory (`M`), or view threads (`H`).
- `lsof`: Lists open files and file descriptors used by processes. Useful options include `lsof -p <pid>` and `lsof +D <dir>`.
- `strace`: Traces system calls and signals. Useful for finding missing files (`ENOENT`).
- `ltrace`: Traces shared library calls made by a process.
- `time` (and `/usr/bin/time`): Measures real, user, and sys time. The `/usr/bin/time` version can also show page fault statistics.
- `renice`: Changes the nice value of a running process (e.g., `renice 20 <pid>`).
- `uptime`: Shows system uptime and load averages for the past 1, 5, and 15 minutes.
- `vmstat`: Reports virtual memory, swap, IO, system, and CPU activity (e.g., `vmstat 2`).
- `iostat`: Reports CPU and I/O statistics for block devices and partitions (e.g., `iostat -p ALL`).
- `iotop`: Interactive tool to view disk I/O usage by process/thread and I/O scheduling class.
- `pidstat`: Monitors and logs resource utilization per process over time.
- `/sys/fs/cgroup`: The pseudo-filesystem interface for cgroups. You interact with files like `cgroup.procs`, `memory.max`, and `cpu.stat`.
