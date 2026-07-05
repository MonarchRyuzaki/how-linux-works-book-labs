# Task 5: Ghost Processes and Cgroups

## Scenario (Mode C - Feynman Interrogation)
Before systemd existed, we used System V init scripts. A common SRE nightmare was the "ghost process" problem: an old backup script would spawn multiple background `rsync` child processes. If you ran `/etc/init.d/backup stop`, the script would terminate, but the orphaned `rsync` processes would keep running in the background, secretly filling up the disk and hogging CPU.

## Your Task
Explain in your own words how `systemd` completely solves this problem. What underlying Linux kernel feature does systemd use to ensure that when you type `systemctl stop backup.service`, every single child process is guaranteed to die? How can you visualize this hierarchy from the command line?

## Solution
Systemd introduces the concept of cgroups, which essentially keeps track of all the forked process that comes out of the main process along with the main processes itself. When we stop the main process, it sees the cgroup the main process is in and stops the whole cgroup. If we do `systemctl status <unit>`, we can see a cgroup section with a tree like structure.

**Correction:**
Exactly right! systemd tags every process with its Control Group (cgroup). Even if a process forks 100 times and daemonizes, the kernel keeps them all in the same cgroup bucket, so systemd can easily round them up and terminate them. 
You can also run `systemd-cgls` to view the entire cgroup tree for the system.
