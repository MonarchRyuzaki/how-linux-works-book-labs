# Task 03: The Read-Only Lockdown

**Mode:** Production Outage Translation (Mode B)
**Calibrated Difficulty:** Junior/Mid SRE
**Target Concepts:** Remounting, `fsck`, Filesystem Corruption

## Scenario
A power outage in the datacenter caused several servers to hard crash. When the database server boots back up, the PostgreSQL service fails to start.

You check the logs and see:
`FATAL: could not open file "pg_xact/0000": Read-only file system`

You run `mount | grep /var/lib/postgresql` and see:
`/dev/sdb1 on /var/lib/postgresql type ext4 (ro,relatime)`

The partition has mounted itself as `ro` (read-only). 

## Your Task
1. Why did the Linux kernel intentionally mount this partition as read-only after a power failure? What is it trying to protect you from?
2. You need to repair the filesystem and get it back to read-write mode. Write out the exact sequence of commands you would use to unmount, check/repair the disk, and remount it safely.

---
### Your Solution:
1. There are likely data inconsistency due to ungraceful shutdown so the kernel mounted it in read-only mode to stop use from using corrupted data and corrupting it any further
2. Here are the sequence of commands
```bash
sudo umount /dev/sdb1

sudo fsck -y /dev/sdb1 [Remove -y for manual inspection]

sudo mount -o remount,rw /data [not sure if this is the exact command cuz it is about remounting but since we unmounted `sudo mount /dev/sdb1 /var/lib/postgresql` might be better. Ofc we will use UUID instead of just the drive name]
```

**Correction:**
Spot on! 
1. **The Why:** You correctly identified that the kernel mounts it as read-only to protect the data from further corruption after an ungraceful shutdown.
2. **The Fix:** Your sequence is practically perfect. You unmount it, run `fsck -y`, and then mount it again. You are absolutely right that if you `umount` it completely, you just run a standard `mount` command rather than a `remount`! (If you couldn't unmount it because it was busy, you would use `mount -o remount,rw /data` after fixing it). Great job noting the UUID best practice too!
