# 1. Partitioning a Drive

### Core Concept
Partitioning is the act of slicing a physical raw disk into smaller, logical boundaries. The Linux kernel treats each partition as an isolated, independent block device.

### MBR vs. GPT
*   **MBR (Master Boot Record):** The legacy standard. Limited to 4 primary partitions. To bypass this, you create an "Extended" partition to hold "Logical" partitions.
*   **GPT (GUID Partition Table):** Modern standard. Supports massive drives and effectively removes the 4-partition limit.

### Essential Commands
```bash
# Safely open fdisk to edit partitions in memory
sudo fdisk /dev/sda
# Inside fdisk: 
# 'p' to print table, 'n' for new partition, 'd' for delete, 'w' to write and exit.

# Instantly create a GPT partition table using parted (DANGEROUS: Live write)
sudo parted /dev/sda mklabel gpt

# Force the kernel to rescan the partition table if it didn't update automatically
sudo partprobe /dev/sda
# OR
sudo blockdev --rereadpt /dev/sda
```

### Kernel Partition Scanning (partprobe)
When you modify a partition table and write it to disk, the Linux kernel must read it to create the corresponding block devices (like `/dev/sda1`). 
*   **Real Hardware:** The kernel automatically scans physical disks (like `/dev/sda`) by default. If you use `fdisk` and hit `w`, the new partitions appear instantly. You **do not** need `partprobe`.
*   **The "Busy" Exception:** The only time you need `partprobe` on physical hardware is if a partition on that disk is currently mounted ("busy"). In this case, `fdisk` will fail to force a reread to avoid crashing the system. You must run `partprobe` to carefully map the new partitions without touching the busy ones.
*   **Loop Devices:** By default, loop devices are treated as single block devices, not partitioned drives. If you partition a loop device, the kernel will ignore the partition table. You must attach the loop device with `losetup -P` to force the kernel to actively scan it just like real hardware.

### Caveats
Never repartition a disk that has actively mounted filesystems! Always unmount first.
