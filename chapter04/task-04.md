# Task 04: Loop Device & Formatting Lab

**Mode:** "Break It For Me" (Mode A - Hands-On Lab)
**Calibrated Difficulty:** Junior/Mid SRE
**Target Concepts:** Loop Devices, Partitioning (`fdisk`), Formatting (`mkfs`), Mounting

## Scenario
Instead of a simulated outage, this is a "DO" task. You need to provision a secure, isolated storage area for a new internal tool, but you don't have any spare physical hard drives. You must simulate a hard drive using a loop device.

## Your Task
Execute the following steps in your terminal, and document the commands you used in the solution block below.
1. Create a 500MB empty file named `virtual_drive.img` using `dd` or `truncate`.
2. Attach this file to a loop device.
3. Use `fdisk` on the loop device to create a single primary partition that takes up the whole space (MBR or GPT, your choice).
4. *Crucial step:* The kernel might not see the new partition on the loop device immediately. Run `partprobe` or `kpartx` so the partition device (e.g., `/dev/loop0p1`) appears.
5. Format the partition with the `ext4` filesystem.
6. Create a directory `/mnt/tool_data` and mount the new partition there.
7. Verify it is mounted correctly with `df -h`.

---
### Your Solution:
1. `truncate -s 500MB virtual_drive.img`
2. ` sudo losetup -P --find --show virtual_drive.img
/dev/loop27
losetup: virtual_drive.img: Warning: file does not fit into a 512-byte sector; the end of the file will be ignored.
`
3. `sudo fdisk /dev/loop27`
`Command (m for help): g
Created a new GPT disklabel (GUID: 140931E6-02CA-5246-8DB7-0667D4A86D1D).
Command (m for help): n
Partition number (1-128, default 1):
First sector (2048-976528, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-976528, default 976528):
Created a new partition 1 of type 'Linux filesystem' and of size 475.8 MiB.
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.`

4. `sudo mkfs.ext4 /dev/loop27p1
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done
Creating filesystem with 121810 4k blocks and 121856 inodes
Filesystem UUID: bd1b3f84-660c-4560-be49-e78fdf860757
Superblock backups stored on blocks:
	32768, 98304
Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done`

5. `sudo mkdir /mnt/tool_data`

6. `sudo mount UUID=bd1b3f84-660c-4560-be49-e78fdf860757 /mnt/tool_data/`

7. `df -h /mnt/tool_data/
Filesystem      Size  Used Avail Use% Mounted on
/dev/loop27p1   430M   24K  396M   1% /mnt/tool_data
`

**Correction:**
Flawless execution! You successfully created a dummy image, attached it with `-P` (which correctly auto-scanned the partition table, saving you from needing `partprobe`), used `fdisk` to create a GPT layout, formatted it with ext4, and correctly mounted it via its UUID. This is exactly how you safely simulate and test storage configurations.
