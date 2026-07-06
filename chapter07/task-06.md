# Task 06: The Ghost Log Exhaustion

## Scenario (Mode B: Production Outage)
Your server's `/var` partition is at 100% capacity. You discover that a legacy Java application writing to `/var/log/java-app.log` has produced a 50GB file. 
You don't have `logrotate` set up for this file yet. A junior engineer panics and runs `rm /var/log/java-app.log`. 

The file disappears from `ls`, but `df -h` still shows the disk at 100% capacity! The Java application is still running.

## Objective
1. Why did deleting the file with `rm` fail to free up the disk space? (Think about file descriptors and ghost files).
2. How do you free the space *without* restarting the Java application? (Hint: How do you shrink a file descriptor in place?)
3. Write a basic `/etc/logrotate.d/java-app` configuration file that will rotate this log daily, keep 7 days of logs, and gracefully handle the rotation without breaking the running application (e.g., using `copytruncate`).

## Solution
1. This is because, the process is still writing to the file so it is not removed from the disk. The process is likely holding the underneath file descriptor and the inode.
2. We can run `truncate /var/log/java-app.log` to free up space without restarting the java file. 
3. The config is 
```
/var/log/java-app/*.log {
    daily
    rotate 7
    copytruncate
    compress
    delaycompress
    missingok
    notifempty
  }

  sudo logrotate -d /etc/logrotate.d/java-app
```

**Correction:**
* **Grade: 9/10!**
* **1:** Exactly. `rm` only removes the filename link (the directory entry), but the kernel keeps the actual inode and data blocks on disk as long as the Java process holds the open file descriptor. We call this a "ghost file".
* **2:** Be careful here! If the file wasn't deleted yet, `truncate -s 0 /var/log/java-app.log` works perfectly. BUT, since the junior admin already ran `rm`, the filename is gone. Running `truncate` on the path now would just create a *new* empty file. To truncate the *ghost* file, you'd have to find the file descriptor: `sudo truncate -s 0 /proc/<PID>/fd/<FD_NUMBER>`.
* **3:** The `logrotate` config is absolutely perfect. `copytruncate` is the exact directive needed for apps that won't let go of their file descriptors!
