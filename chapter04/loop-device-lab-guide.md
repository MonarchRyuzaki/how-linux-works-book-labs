# Loop Device Lab Guide: Safe Storage Practice

This guide provides the exact workflow to safely simulate physical hard drives, partition them, format them, configure LVM, and cleanly destroy them—all without touching your actual host OS storage.

---

### Step 1: Provisioning "Fake" Hard Drives
Create large, empty sparse files. These take up almost zero physical space initially but logically appear as 1GB disks to the kernel.
```bash
truncate -s 1G disk1.img
truncate -s 1G disk2.img
```

### Step 2: Attaching the Loop Devices
Attach the dummy files to loop pseudo-devices. 
*Note: The `-P` (Partscan) flag is critical! By default, loop devices do not actively scan for partitions. Using `-P` forces the kernel to treat the loop device exactly like a real physical hard drive. Because of this flag, you do not need to run `partprobe` after using `fdisk`.*
```bash
sudo losetup -P --find --show disk1.img
sudo losetup -P --find --show disk2.img
```
*(Take note of the output, e.g., `/dev/loop0` and `/dev/loop1`)*

---

### Step 3: Practicing (The Lab Phase)
Now you can safely run any dangerous SRE commands on those loop devices:

**A. Standard Partitioning & Formatting:**
```bash
sudo fdisk /dev/loop0
sudo mkfs.ext4 /dev/loop0p1
sudo mount /dev/loop0p1 /mnt/test
```

**B. LVM Matrix (Logical Volume Manager):**
```bash
sudo pvcreate /dev/loop0 /dev/loop1
sudo vgcreate my_pool /dev/loop0 /dev/loop1
sudo lvcreate -L 500M -n my_vol my_pool
sudo mkfs.xfs /dev/my_pool/my_vol
```

---

### Step 4: Graceful Cleanup (CRITICAL)
You must tear down the storage stack in reverse order to release the kernel locks. If you skip this, `losetup -d` will fail with a "Device is busy" error.

**1. Unmount any active filesystems:**
```bash
sudo umount /mnt/test
sudo umount /dev/my_pool/my_vol
```

**2. Tear down the LVM Matrix (Top to Bottom):**
```bash
sudo lvremove /dev/my_pool/my_vol  # (Press 'y' to confirm)
sudo vgremove my_pool
sudo pvremove /dev/loop0 /dev/loop1
```
*(Note: If you didn't use LVM, you can skip this step).*

**3. Detach the loop devices:**
```bash
sudo losetup -d /dev/loop0 /dev/loop1
```

**4. Delete the physical dummy files:**
```bash
rm disk1.img disk2.img
```

You are now back to a perfectly clean host system!
