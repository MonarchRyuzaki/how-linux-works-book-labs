### Prerequisite Concepts

Before diving into the chapter's mechanics, let's establish some foundational terms that the author assumes you are familiar with:

*   **Block Devices** `[Context Added]`: In Linux, hardware is represented as files. A "block device" (like `/dev/sda`) represents a storage drive that reads and writes data in fixed-size chunks (blocks). 
*   **User Space vs. Kernel Space** `[Context Added]`: The kernel has direct, privileged access to the physical hard drive hardware. Tools like `fdisk` and `parted` run in "user space" (unprivileged). When a user space tool needs to alter the disk, it must ask the kernel to do the heavy lifting via a **System Call**.
*   **File Descriptors / Inodes** `[Context Added]`: (Recalling your cheatsheet) Remember that the OS uses inodes to track where physical data lives. Filesystems are the databases that map human-readable names (like `log.txt`) to these physical inodes. 

---

### Deep Dive: Disks, Partitions, and Filesystems

#### 1. The Anatomy of a Drive
The book illustrates a schematic of a disk. It is crucial to understand the layers of abstraction from bottom to top:
1.  **The Physical Disk**: Represented by `/dev/sda` or `/dev/nvme0n1`.
2.  **The Partition Table**: A tiny map written at the very beginning of the disk that tells the OS how the disk is sliced up.
3.  **The Partitions**: The logical slices defined by the table (e.g., `/dev/sda1`, `/dev/sda2`). The kernel treats these slices as if they were entirely separate hard drives.
4.  **The Filesystem**: The database (like `ext4`, `xfs`, `FAT32`) installed *inside* the partition. It dictates how directories and files are organized.

#### 2. Partition Tables: MBR vs. GPT
The chapter introduces the two dominant ways to map out a disk:
*   **MBR (Master Boot Record / "msdos")**: The legacy standard from the 1980s. 
    *   **The Trap**: It has a hard limit of **4 primary partitions**.
    *   **The Workaround**: If you need 5 partitions, you must create 3 "Primary" partitions, and make the 4th one an "Extended" partition. The Extended partition acts as a container to hold multiple "Logical" partitions inside it.
*   **GPT (GUID Partition Table)**: The modern standard. It supports massive drives and effectively removes the 4-partition limit, meaning everything can just be a normal partition. 

#### 3. The Great Divide: `fdisk` vs `parted`
The author heavily focuses on the difference in how the two primary partitioning tools interact with the Linux Kernel. As an SRE, understanding this difference prevents catastrophic data loss.

*   **`fdisk` (The Safe Way)**
    *   **Mechanism**: It is interactive. When you delete or create partitions, it happens entirely in RAM (user space). **No changes are made to the physical disk** until you type `w` (write) and exit. 
    *   **Kernel Interaction**: When you exit, `fdisk` makes a *single system call* telling the kernel: "I updated the map, please reread the whole disk."
*   **`parted` (The Live Way)**
    *   **Mechanism**: It operates live. The moment you press enter to delete a partition, **it is instantly wiped from the disk**. There is no undo button. 
    *   **Kernel Interaction**: It doesn't tell the kernel to reread the whole disk. Instead, it signals the kernel piece-by-piece as you make changes. 

#### 4. Kernel Desync & Troubleshooting
Sometimes, you change the partition table, but the OS doesn't see the new partitions (e.g., you create `/dev/sdd2`, but running `ls /dev/sd*` doesn't show it). This happens because the kernel's memory is out of sync with the physical disk map.
*   **The Fix**: You can manually trigger the system call that `fdisk` uses by running:
    ```bash
    sudo blockdev --rereadpt /dev/sdd
    ```
    This forces the kernel to look at the disk, read the new table, and dynamically generate the new `/dev/sdd2` block device file for you to use.

---

> **MANDATORY DISCLAIMER:** Please read the original chapter! This summary builds a mental scaffold, but the original author often buries subtle insights, specific code edge-cases, and trade-offs in their phrasing that summaries cannot capture. Theory is best cemented by reading the original text now that you have the map of what to look for.
