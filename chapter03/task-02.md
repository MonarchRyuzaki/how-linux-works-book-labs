# Task 02: Surgical Precision with dd

## Scenario (Mode A/B Blended: Break & Fix)
A developer accidentally wiped the partition table on a secondary disk (`/dev/sdb`) by running a bad script. The disk is now unreadable. 

Fortunately, you have a binary backup of the disk's Master Boot Record (the first 512 bytes), saved as `mbr_backup.bin`. You need to restore *only* those first 512 bytes back to `/dev/sdb`. 

If you mess up and write too much data, you will overwrite the actual data partitions!

## Goal
Write the exact `dd` command to copy exactly 512 bytes from `mbr_backup.bin` to `/dev/sdb`. Explain what the `if`, `of`, `bs`, and `count` parameters do in your command.

## Solution
1. `dd if=mbr_backup.bin of=/dev/sdb bs=512 count=1`. The parameters if means input file, of means output file, bs means block size, count means the number of blocks to copy.

**Correction:**
**Grade: 100% (Perfect Execution)**

Spot on! This command is surgical because:
- `bs=512` tells `dd` to read/write in a chunk of 512 bytes.
- `count=1` tells it to stop immediately after 1 chunk.
If you forgot `count=1`, `dd` would keep copying until it reached the end of `mbr_backup.bin`. If the backup file had trailing garbage, it would overwrite your actual partitions on `/dev/sdb`! This is why `dd` has the nickname "Disk Destroyer" if you aren't careful. Excellent job explaining the flags.
