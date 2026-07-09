# Task 1: The Invisible Disk Eater
**Concept:** `lsof`, Ghost Files, Inodes
**Difficulty:** Junior SRE

## The Scenario (Mode B)
You are on-call for the JPMC Settlement platform. The monitoring dashboard turns red: the primary application server has thrown a `no space left on device` alert on the `/tmp` partition. 
You run `df -h` and see that `/tmp` is full. However, when you run `du -sh /tmp/*`, it shows only a few Megabytes of data. 

A ghost file is haunting the server. A process has deleted a massive log file, but it's still holding the file descriptor open, meaning the kernel hasn't freed the disk space. 

## The Setup (Mode A)
Run the following script to simulate the outage:
```bash
cat << 'EOF' > /tmp/outage-sim.sh
#!/bin/bash
echo "Starting application server..."
# Create a large dummy file
dd if=/dev/zero of=/tmp/app-error.log bs=1M count=300 status=none
# Keep the file descriptor open via a background loop
exec 3<>/tmp/app-error.log
while true; do
  echo "ERROR: DB timeout" >&3
  sleep 5
done &
APP_PID=$!
# Delete the file to hide it
rm -f /tmp/app-error.log
echo "Application running with PID: $APP_PID"
echo "Oh no, a rogue log-rotation script deleted the log file!"
EOF
chmod +x /tmp/outage-sim.sh
/tmp/outage-sim.sh
```

## The Mission
1. Find the ghost file using `lsof`.
2. Reclaim the disk space **WITHOUT** killing the application process (PID). Hint: Use `truncate` on the exact file descriptor in `/proc/<pid>/fd/<num>`.
3. Verify the space is reclaimed.

## Solution
ps -aux | grep outage
ryuzaki    22892  0.0  0.0  10108  2152 pts/1    S    19:44   0:00 /bin/bash /tmp/outage-sim.sh

lsof -p 22892 | grep deleted
COMMAND     PID    USER   FD   TYPE DEVICE  SIZE/OFF    NODE NAME
outage-si 22892 ryuzaki    3u   REG  259,5 314572800 2363652 /tmp/app-error.log (deleted)

sudo du -sh -L /proc/22892/fd/3
301M	/proc/22892/fd/3

truncate -s 0 /proc/22892/fd/3

sudo du -sh -L /proc/22892/fd/3
0	/proc/22892/fd/3

**Correction:**
Spot on! You found the ghost file and properly reclaimed the disk space using `truncate -s 0`. This is a classic SRE lifesaver that lets you fix a "disk full" error in production without having to restart the application (which would cause downtime). Great job using the `-L` flag on `du` to dereference the symlink!
