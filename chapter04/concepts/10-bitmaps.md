# 10. Bitmap Block Allocation

### Core Concept
When you write a new file, how does the filesystem know which physical blocks on the massive 1TB hard drive are free? 
It doesn't scan the whole drive. Instead, it maintains a **Block Bitmap**.

A bitmap is a long string of binary zeros and ones stored at the beginning of the filesystem. Each bit represents one physical data block on the disk.
*   `0` = Block is free.
*   `1` = Block is currently holding data.

When you delete a file, the OS doesn't erase the physical data on the disk. It simply flips the 1s back to 0s in the Block Bitmap. The data remains there (ghost data) until a new file overwrites it.

### Essential Commands
```bash
# Check block space usage (powered by the Block Bitmap)
df -h

# Check Inode usage (powered by the Inode Bitmap)
df -i

# Advanced: View the location of the block and inode bitmaps inside an ext4 filesystem
sudo tune2fs -l /dev/sda1 | grep "Bitmap"
```

### Caveat (The Inode Trap)
There is also an **Inode Bitmap** that tracks free Inodes. Even if the Block Bitmap says you have 500GB of free space (checked via `df -h`), if the Inode Bitmap is entirely 1s because you created millions of 0-byte files (checked via `df -i`), you will get a "No space left on device" error!
