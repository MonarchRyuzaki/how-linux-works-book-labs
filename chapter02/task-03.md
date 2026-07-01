# Task 03: The Phantom File Eating the Disk

## Scenario (Junior/Mid SRE Level)
A critical production server is triggering alerts: `/var` is 100% full. 
A frantic developer logged in, saw that `/var/log/application.log` was 50GB, and quickly ran:
```bash
rm /var/log/application.log
```

However, the alert hasn't cleared. You run `df -h` and `/var` is STILL at 100% usage. `ls -l /var/log/` shows the file is completely gone. 

## The Challenge
1. **The Inode Illusion**: Why did `rm` fail to free up the physical disk space? Explain what the Linux kernel is doing with the file descriptor and the application that is still running.
Solution: I think rm actually just removed the file and did not free up disk space. The process that was reading and writing to the file still hold the underlying Inode. Feel free to correct my explaination.

2. **The Investigation**: How do you find the process that is still holding onto this "ghost" file? (Hint: you learned about looking for open file descriptors).
Solution: If I remember correctly there is a command to look for the process using a file, which is fuser i think. 

3. **The SRE Fix**: Without stopping the running application (which would cause a production outage), how can you force the kernel to drop the physical size of that ghost file to 0 bytes? (This connects to a trick you have in your cheatsheet!).
Solution: We use the truncate command, to delete the contents of the file and put the cursor back to first position. That way we dont lose any data, we can simply move it elsewhere and truncate it to start from 0.

**Correction:**
1. Exactly right! `rm` only removes the filename link (the directory entry), but the physical Inode remains allocated because the application still has an open file descriptor pointing to it.
2. Yes, `lsof` (List Open Files) or `fuser` are the correct tools!
3. Close! You can't just `truncate` the filename anymore because `rm` deleted the name! Instead, you have to truncate it directly via the kernel's process interface: `sudo truncate -s 0 /proc/<PID>/fd/<FD_NUMBER>`. This instantly frees the disk space without breaking the application's stream!

## Related Concepts (Chapter 2)
- `rm` and File linking
- Process listing (`ps`)
- File descriptors and Data streams
