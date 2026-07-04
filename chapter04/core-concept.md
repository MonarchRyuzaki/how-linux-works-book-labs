# Chapter 4: Disks and Filesystems

## Structural Outline
- 4.1 Partitioning Disk Devices
  - 4.1.1 Viewing a Partition Table
  - 4.1.2 Modifying Partition Tables
  - 4.1.3 Creating a Partition Table
- 4.2 Filesystems
  - 4.2.1 Filesystem Types
  - 4.2.2 Creating a Filesystem
  - 4.2.3 Mounting a Filesystem
  - 4.2.4 Unmounting a Filesystem
  - 4.2.5 Checking and Repairing Filesystems
- 4.3 Swap Space
- 4.4 Logical Volume Manager (LVM)
  - 4.4.1 Logical Volume Concepts (PV, VG, LV)
  - 4.4.2 Creating LVM Volumes
  - 4.4.3 Resizing LVM Volumes

## Core Concepts

**4.1 Partitioning Disk Devices**
Disks are subdivided into partitions to organize data. The partition table (MBR or GPT) defines these boundaries. MBR supports up to four primary partitions (with extended/logical partitions for more), while GPT supports large modern drives with virtually unlimited partitions. Tools like `parted` modify partitions live on the disk, whereas `fdisk` stages changes safely in memory before writing.

**4.2 Filesystems**
A filesystem is the database that maps directories and file names to actual physical data blocks (using inodes). Ext4 and XFS are common. Formatting a partition (e.g., `mkfs.ext4`) builds this structure. 

**4.2.3 Mounting & UUIDs (🚨 Critical SRE Best Practice)**
Never mount partitions using hardware device names (e.g., `/dev/sda1`) in `/etc/fstab` because these names can shift on reboot if drives are added/removed. Always mount using the filesystem's UUID (Universally Unique Identifier), which is permanently stamped into the metadata.

**4.2.5 Filesystem Repair & Remounting**
If the kernel detects filesystem corruption (like after a sudden power loss), it will often force the filesystem to mount as Read-Only (`ro`) to prevent further data destruction. You must unmount it and use `fsck` to repair the corruption before remounting it as Read-Write (`rw`).

**4.3 Swap Space**
Swap acts as emergency overflow memory on the hard drive. If the system RAM runs out, inactive memory pages are moved to Swap. You can allocate dedicated swap partitions or use swapfiles on the root filesystem to quickly add capacity without repartitioning.

**4.4 Logical Volume Manager (LVM)**
LVM decouples filesystems from physical disks, allowing you to resize volumes live. Physical Volumes (PVs) are pooled into a Volume Group (VG), from which you can carve out Logical Volumes (LVs). If a disk fills up, you can simply add a new disk to the VG and expand the LV without unmounting or rebooting.

## Commands & Utilities
- `fdisk` / `parted`: Used to manage partition tables on block devices.
- `lsblk`: Lists block devices, their partitions, and mount points in a tree format. Use `-f` to see UUIDs.
- `mkfs`: Makes a filesystem (e.g., `mkfs.ext4`, `mkfs.xfs`).
- `mount` / `umount`: Attaches and detaches filesystems to the directory tree.
- `fsck`: Checks and repairs a Linux filesystem.
- `pvcreate` / `vgcreate` / `lvcreate`: Commands to build the LVM stack.
- `lvextend`: Expands a logical volume. Must be followed by `resize2fs` (for ext4) or `xfs_growfs` (for XFS) to expand the actual filesystem inside it.
- `mkswap` / `swapon`: Formats a file or partition as swap space and enables it.
