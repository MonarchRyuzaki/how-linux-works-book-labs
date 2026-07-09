# Task 3: The Noisy Neighbor
**Concept:** CPU Scheduling, `renice`, `top`
**Difficulty:** Junior SRE

## The Scenario (Mode B)
Other developers are complaining that the main build server (`jump-host-01`) is incredibly laggy. Typing in SSH has a 2-second delay. You check `uptime` and the load average is spiking. Someone has launched a massive, poorly optimized batch process in the background. You can't kill it because it's generating an important report for the CTO, but you need to prioritize the SSH interactive sessions so developers can work.

## The Setup (Mode A)
Run this command to simulate the CPU hog:
```bash
bash -c 'while true; do :; done' &
```

## The Mission
1. Use `top` to identify the PID of the process consuming 100% of a CPU core.
2. Use the `renice` command to lower the priority (increase the "nice" value) of this rogue process to `19`.
3. Observe the `NI` and `PR` columns in `top` to verify the change. 
4. Clean up by killing the process when you're done.

## Solution
bash -c 'while true; do :; done' &
[1] 29388
1. Output of top command 
29388 ryuzaki   20   0    9976   3628   3348 R 100.0   0.0   0:24.18 bash
2. renice -n 19 -p 29388
29388 (process ID) old priority 0, new priority 19
3.  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
  29388 ryuzaki   39  19    9976   3628   3348 R 100.0   0.0   2:50.12 bash
4.  kill 29388
[1]+  Terminated              bash -c 'while true; do :; done'


**Correction:**
Flawless. You correctly identified the PID and used `renice` to shove it into the background (NI=19). This tells the kernel's scheduler to only give it CPU cycles when no other process (like your SSH session) needs them.
