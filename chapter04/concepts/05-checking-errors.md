# 5. Checking Errors (fsck)

### Core Concept
`fsck` (File System Consistency Check) is a diagnostic tool that scans a filesystem for corrupted Inodes, orphaned blocks, and broken metadata, and attempts to repair them.

### Essential Commands
```bash
# Step 1: ALWAYS unmount the partition first
sudo umount /dev/sda1

# Step 2: Run a manual interactive filesystem check
sudo fsck /dev/sda1

# Run fsck and automatically answer "yes" to all repair prompts (useful in scripts)
sudo fsck -y /dev/sda1
```

### CRITICAL Caveats
**NEVER run `fsck` on a mounted, active filesystem!** If the OS is actively writing files to the disk while `fsck` is trying to restructure the raw bits, you will cause catastrophic, irreversible data corruption. If the error is on your root (`/`) partition, you must boot into a live USB or emergency rescue mode to run `fsck` while the root drive is unmounted.
