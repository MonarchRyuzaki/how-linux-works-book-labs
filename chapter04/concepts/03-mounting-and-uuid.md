# 3. Mounting and the UUID Rule

### Core Concept
Unlike Windows (which uses C: or D: drives), Linux has a single unified directory tree starting at `/`. Mounting is the process of attaching a formatted partition's filesystem to a specific empty folder (the mount point) in that tree.

### The UUID Rule (SRE Best Practice)
Never mount by hardware device names (e.g., `/dev/sda1`). Device names are assigned dynamically on boot. If you plug in a USB drive, your main drive `/dev/sda` might become `/dev/sdb`, causing your server to crash on boot. Always use the **UUID**.

### Essential Commands
```bash
# Find the UUID of your partition
sudo blkid
# OR
lsblk -f

# Mount the partition manually using the UUID
sudo mount UUID="1234abcd-56ef-78gh" /mnt/data

# Unmount the partition
sudo umount /mnt/data
```

### Persistent Mounting (`/etc/fstab`)
To make a mount survive a reboot, add it to `/etc/fstab`:
```text
# Example /etc/fstab entry
UUID=1234abcd-56ef-78gh   /mnt/data   ext4   defaults   0   2
```
