# Chapter 3: Devices

## Structural Outline
- 3.1 Device Files
- 3.2 The sysfs Device Path
- 3.3 dd and Devices
- 3.4 Device Name Summary
  - 3.4.1 Hard Disks: /dev/sd*
  - 3.4.2 Virtual Disks: /dev/xvd*, /dev/vd*
  - 3.4.3 Non-Volatile Memory Devices: /dev/nvme*
  - 3.4.4 Device Mapper: /dev/dm-*, /dev/mapper/*
  - 3.4.5 CD and DVD Drives: /dev/sr*
  - 3.4.6 PATA Hard Disks: /dev/hd*
  - 3.4.7 Terminals: /dev/tty*, /dev/pts/*, and /dev/tty
  - 3.4.8 Serial Ports: /dev/ttyS*, /dev/ttyUSB*, /dev/ttyACM*
  - 3.4.9 Parallel Ports: /dev/lp0 and /dev/lp1
  - 3.4.10 Audio Devices: /dev/snd/*, /dev/dsp, /dev/audio, and More
  - 3.4.11 Device File Creation
- 3.5 udev
  - 3.5.1 devtmpfs
  - 3.5.2 udevd Operation and Configuration
  - 3.5.3 udevadm
  - 3.5.4 Device Monitoring
- 3.6 In-Depth: SCSI and the Linux Kernel
  - 3.6.1 USB Storage and SCSI
  - 3.6.2 SCSI and ATA
  - 3.6.3 Generic SCSI Devices
  - 3.6.4 Multiple Access Methods for a Single Device

## Core Concepts

**3.1 Device Files**
Linux interfaces with hardware devices by presenting them as files within the `/dev` directory. These can be block devices (accessed in fixed chunks, like hard drives), character devices (accessed as a data stream, like `/dev/null`), pipes, or sockets.

**3.2 The sysfs Device Path**
While `/dev` provides an interface to use devices, the `/sys` directory (sysfs) provides a uniform view of devices based on actual hardware attributes. It exports detailed kernel data about devices and drivers to user space.

**3.3 dd and Devices**
The `dd` tool is essential for manipulating block and character devices directly. It reads and writes data in fixed-size blocks, making it powerful for copying raw device data or zeroing out drives.

**3.4 Device Name Summary**
Linux relies on consistent naming conventions for hardware. For example, SCSI/SATA hard disks are prefixed with `sd`, virtual disks with `vd` or `xvd`, and NVMe drives with `nvme`. Terminals are represented by `tty` devices.

**3.5 udev**
The kernel doesn't maintain static `/dev` files anymore. Instead, the kernel notifies a user-space daemon (`udevd`) when a device is added or removed. The `udev` system uses a set of rules (in `/lib/udev/rules.d/`) to automatically create dynamic device nodes, set permissions, and manage symbolic links like `/dev/disk/by-id/`.

**3.6 In-Depth: SCSI and the Linux Kernel**
The Linux kernel uses the SCSI subsystem as a foundational layer for managing storage. Even USB flash drives and modern SATA drives utilize SCSI command translation (via drivers like `libata`) to communicate with the kernel, highlighting the modular and layered architecture of Linux device drivers.

## Commands & Utilities
- `dd`: A powerful utility for copying and converting raw data across files and devices. Common options include `if` (input file), `of` (output file), and `bs` (block size).
- `ls -l /dev`: Lists device files. The first character of the output denotes if the device is a block (`b`), character (`c`), pipe (`p`), or socket (`s`) device.
- `udevadm`: An administration tool for the udev daemon. Useful for querying device paths (`udevadm info`) and monitoring device events in real-time (`udevadm monitor`).
- `lsscsi`: Lists attached SCSI devices (which includes SATA and USB storage drives on modern Linux), showing their addresses and block device names.
- `mknod`: A legacy command used to manually create block or character device files, requiring major and minor device numbers.
- `chvt`: A command to force a switch between virtual terminals (e.g., `chvt 1`).
