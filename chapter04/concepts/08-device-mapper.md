# 8. The Device Mapper

### Core Concept
The Device Mapper is a framework in the Linux kernel that maps physical block devices onto higher-level, virtual block devices. It is the underlying engine that makes LVM, disk encryption (LUKS), and software RAID possible.

### Essential Commands & Observation
When you create a Logical Volume in LVM, the device mapper creates a virtual device map. You can observe how Linux links them:
```bash
# List all device mapper symlinks
ls -l /dev/mapper/

# You will see the LVM names symlinked to the raw internal dm-X names:
# lrwxrwxrwx 1 root root 7 Jul 3 20:00 /dev/mapper/my_pool-my_vol -> ../dm-0

# View the low-level kernel device mapper table
sudo dmsetup ls
```

### Caveat
If you run tools like `lsblk`, you might see the volume listed as `/dev/dm-0` or `/dev/dm-1`. These are the internal, raw device mapper IDs. **Never use the `dm-X` names in scripts or `/etc/fstab`**. They are not guaranteed to persist across reboots. Always use the `/dev/mapper/...` path, the `/dev/vgname/lvname` path, or ideally, the UUID.
