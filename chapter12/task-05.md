# Task 05: The Hanging Mount (SSHFS & FUSE)

**Scenario:**
You used `sshfs` to mount a remote developer's codebase to your local machine to run some grep searches:
```bash
sshfs dev@10.0.5.50:/home/dev/project /mnt/remote_code
```

The remote server rebooted unexpectedly. Now, your `/mnt/remote_code` directory is completely hung. If you run `ls /mnt/remote_code`, your terminal freezes forever waiting for the dead SSH connection.

You try to unmount it as the superuser using the standard command:
```bash
sudo umount /mnt/remote_code
```
It fails, saying the device is busy or inaccessible.

**Your Objective:**
Because SSHFS is a FUSE (File System in User Space) mount, how should you properly unmount it as the regular user?

### Solution
1. We can run `fusermount -u /mnt/remote_code`

**Correction:**
**Grade: 100%**
Perfect. Because it was mounted in user-space via FUSE, the standard kernel `umount` often hangs or complains. `fusermount -u` correctly detaches the hung user-space process.
