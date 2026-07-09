# Task 5: The Thread Bomb
**Concept:** Threads, `ps m`, `top -H`
**Difficulty:** Junior SRE

## The Scenario (Mode B)
You receive an alert that the number of active tasks on a server is dangerously high. You run `ps aux | wc -l` and it shows only about 40 processes running. However, `uptime` and the system metrics indicate there are over 500 tasks being scheduled by the kernel. 
An application is spawning a massive number of threads under a single process!

## The Setup (Mode A)
Run this Python script to simulate a multithreaded application:
```bash
cat << 'EOF' > /tmp/thread-bomb.py
import threading
import time

def worker():
    while True:
        time.sleep(10)

for i in range(100):
    t = threading.Thread(target=worker)
    t.daemon = True
    t.start()

print("Application running with 100 threads...")
while True:
    time.sleep(1)
EOF
python3 /tmp/thread-bomb.py &
```

## The Mission
1. Standard `top` and `ps aux` hide threads by default. 
2. Find a way to view all the threads (TIDs) of this python process using either `top` (hint: press `H`) or `ps` (hint: use the `m` flag).
3. Find the Process ID (PID) and how many threads it currently has.
4. Kill the process.

## Solution
1. ps auxm 
ryuzaki    33178  0.0  0.0 7065536 11224 pts/1   -    20:18   0:00 python3 /tmp/thread-bomb.py
ryuzaki        -  0.0    -      -     - -        Sl   20:18   0:00 -
100*

2.  ps -o nlwp 33178
NLWP
 101

3.  kill 33178
[1]+  Terminated              python3 /tmp/thread-bomb.py


**Correction:**
Outstanding! Not only did you use the `m` flag with `ps`, but you also pulled out the exact number of Light Weight Processes (`NLWP`) to confirm the thread count (1 main + 100 workers). Very strong CLI knowledge.
