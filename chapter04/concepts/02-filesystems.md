# 2. Installing a Filesystem

### Core Concept
A raw partition is just empty space. A filesystem (like `ext4`, `xfs`, or `btrfs`) is the actual database structure that organizes raw bits into a hierarchy of directories and files. "Formatting" a drive means installing this filesystem database onto the partition.

### Essential Commands
```bash
# Format a partition as ext4
sudo mkfs.ext4 /dev/sda1

# Format a partition as xfs (common in RedHat/CentOS)
sudo mkfs.xfs /dev/sda1

# View the filesystem types and UUIDs of all formatted partitions
lsblk -f
```

### Caveats & Warnings
*   **Destructive Action:** Running `mkfs` on a partition instantly wipes out any existing metadata and effectively destroys all previous data.
*   **UUID Generation:** The moment you run `mkfs`, the tool generates a permanent, randomized serial number (UUID) and stamps it into the partition. This UUID is how you should always identify the drive going forward.
