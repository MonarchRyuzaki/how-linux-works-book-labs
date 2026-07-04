# 9. Inodes and Hardlinks

### Core Concept
In Linux filesystems, the filename is completely separate from the actual data. 
*   **Inode:** The true "file" database entry. It holds the permissions, ownership, timestamps, and the exact physical disk blocks where the data lives.
*   **Filename:** A simple sticky note (a "link") that pointing to an Inode number.

### Essential Commands
```bash
# View the Inode number of a file (the first column of output)
ls -li original.txt

# Create a hardlink (two filenames pointing to the exact same Inode)
ln original.txt hardlink.txt

# Verify they have the same Inode number
ls -li original.txt hardlink.txt

# Check overall Inode usage on a filesystem
df -i
```

### Hardlink Mechanics & Caveats
*   If you edit `hardlink.txt`, you are editing the exact same physical disk blocks as `original.txt`.
*   If you delete `original.txt`, the data is perfectly safe and accessible via `hardlink.txt`. The disk space is only freed when the *last* hardlink is deleted.
*   **Caveat:** Hardlinks **cannot cross filesystems**. An Inode number is only unique within its own partition. You cannot hardlink a file on `/dev/sda1` to a directory on `/dev/sdb1`. (You must use symlinks `ln -s` for that).
