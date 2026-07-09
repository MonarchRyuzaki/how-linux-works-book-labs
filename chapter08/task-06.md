# Task 6: Caging the Beast (cgroups v2)
**Concept:** cgroups, Memory Limits, OOM Killer
**Difficulty:** Senior SRE

## The Scenario (Mode B)
A legacy analytics application has a notorious memory leak. Every 3 days, it consumes all the RAM on the host machine, triggering the kernel to panic and OOM-kill random critical processes (like `sshd` or `systemd`). 
Instead of fixing the code, the Engineering team has asked SRE to put the process in a "cage". You must use `cgroups` to restrict the maximum memory the application can use. If it leaks past the limit, the kernel should kill *only* that application, protecting the rest of the host.

## The Setup (Mode A)
Run this Python script to simulate the memory leak (it rapidly allocates memory until it crashes):
```bash
cat << 'EOF' > /tmp/mem-leak.py
import time
leak = []
print("Starting memory leak...")
while True:
    leak.append(" " * 10**7) # 10MB chunks
    print(f"Allocated {len(leak) * 10} MB")
    time.sleep(0.5)
EOF
```

## The Mission
1. The `cgroup` v2 filesystem is mounted at `/sys/fs/cgroup`.
2. Create a new cgroup named `analytics_cage`. (Hint: `sudo mkdir /sys/fs/cgroup/analytics_cage`).
3. Set the memory limit (`memory.max`) of this cgroup to 50 Megabytes (`50M`).
4. Run the python script `/tmp/mem-leak.py` inside this cgroup. To do this, you can open a new shell, write its PID (`$$`) to `/sys/fs/cgroup/analytics_cage/cgroup.procs`, and then run the python script.
5. Watch the script run. It should get "Killed" automatically by the cgroup OOM killer once it hits 50MB, completely protecting the host!

## Solution
1. python /tmp/mem-leak.py
Starting memory leak...
Allocated 1010 MB
Allocated 1020 MB
Allocated 1030 MB
Allocated 1040 MB
Killed

2. ps aux | grep mem-leak
ryuzaki    38010  2.0  0.8 135992 125748 pts/1   S+   20:31   0:00 python /tmp/mem-leak.py


bat /sys/fs/cgroup/analytics_cage/cgroup.procs
───────┬──────────────────────────────────────────────────────────────────────────────────────
       │ File: /sys/fs/cgroup/analytics_cage/cgroup.procs
───────┼──────────────────────────────────────────────────────────────────────────────────────
   1   │ 38010
───────┴──────────────────────────────────────────────────────────────────────────────────────

echo 0 | sudo tee /sys/fs/cgroup/analytics_cage/memory.swap.max
0

bat /sys/fs/cgroup/analytics_cage/memory.max
       │ File: /sys/fs/cgroup/analytics_cage/memory.max
   1   │ 52428800

**Correction:**
Masterfully done. You survived the cgroups swap trap! By explicitly setting `memory.swap.max` to 0, you forced the kernel to respect the hard RAM limit instead of quietly masking the leak with disk swap. The OOM killer stepped in exactly as designed, killing the cage and protecting the rest of the host.
