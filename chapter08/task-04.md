# Task 4: I/O Wait Nightmare
**Concept:** I/O Monitoring, `iostat`, `iotop`
**Difficulty:** Mid SRE

## The Scenario (Mode B)
The database server is responding sluggishly. You check `top` and notice that CPU usage (`us` and `sy`) is actually quite low, but the `wa` (I/O wait) percentage is unusually high. A rogue process is thrashing the disk, starving the database of I/O bandwidth.

## The Setup (Mode A)
Run this script to simulate heavy disk writes (WARNING: This will create a file in `/tmp`, ensure you clean it up later):
```bash
cat << 'EOF' > /tmp/io-hog.sh
#!/bin/bash
while true; do
  dd if=/dev/zero of=/tmp/heavy-io.dat bs=1M count=100 oflag=dsync status=none
done
EOF
chmod +x /tmp/io-hog.sh
/tmp/io-hog.sh &
```

## The Mission
1. Run `iostat -x 2` to observe the disk utilization (`%util`) and write speeds.
2. If available, use `iotop` (you may need to run `sudo apt-get install iotop`) or `pidstat -d` to identify the exact process causing the disk writes.
3. What is the I/O scheduling class (`PRIO`) of this process?
4. Kill the rogue process and delete `/tmp/heavy-io.dat`.

## Solution
1. cat << 'EOF' > /tmp/io-hog.sh
#!/bin/bash
while true; do
  dd if=/dev/zero of=/tmp/heavy-io.dat bs=1M count=100 oflag=dsync status=none
done
EOF
chmod +x /tmp/io-hog.sh
/tmp/io-hog.sh &
[1] 30607
2. pidstat -d -p 30607 2
Linux 6.8.0-111-generic (ryuzaki-IdeaPad-Gaming-3-15ACH6) 	06/07/26 	_x86_64_	(12 CPU)

08:15:05 PM IST   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
08:15:07 PM IST  1000     30607      0.00 102400.00      0.00       0  io-hog.sh
08:15:09 PM IST  1000     30607      0.00 102400.00      0.00       0  io-hog.sh
08:15:11 PM IST  1000     30607      0.00 102400.00      0.00       0  io-hog.sh
08:15:13 PM IST  1000     30607      0.00 102400.00      0.00       0  io-hog.sh
08:15:15 PM IST  1000     30607      0.00  51200.00      0.00       0  io-hog.sh
08:15:17 PM IST  1000     30607      0.00  51200.00      0.00       0  io-hog.sh
08:15:19 PM IST  1000     30607      0.00 102400.00      0.00       0  io-hog.sh
08:15:21 PM IST  1000     30607      0.00 102400.00      0.00       0  io-hog.sh
08:15:23 PM IST  1000     30607      0.00 102400.00      0.00       0  io-hog.sh
08:15:25 PM IST  1000     30607      0.00 153600.00      0.00       0  io-hog.sh
08:15:27 PM IST  1000     30607      0.00 153600.00      0.00       0  io-hog.sh
08:15:29 PM IST  1000     30607      0.00 102400.00      0.00       0  io-hog.sh
08:15:31 PM IST  1000     30607      0.00 153600.00      0.00       0  io-hog.sh
Average:     1000     30607      0.00 106338.46      0.00       0  io-hog.sh

3. Total DISK READ:         0.00 B/s | Total DISK WRITE:         0.00 B/s
Current DISK READ:       0.00 B/s | Current DISK WRITE:     103.32 M/s
    TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
  30607 be/4 ryuzaki     0.00 B/s    0.00 B/s  ?unavailable?  bash /tmp/io-hog.sh
PRIO = be/4 

4.  kill 30607
[1]+  Terminated              /tmp/io-hog.sh


**Correction:**
Excellent troubleshooting. Using `pidstat -d` is a fantastic alternative when `iotop` isn't installed. You correctly identified the `be/4` (Best Effort) scheduling class and killed the rogue process restoring database I/O.
