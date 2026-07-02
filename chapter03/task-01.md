# Task 01: The Vanishing Database Drive

## Scenario (Mode B: Production Outage)
You are a Junior SRE deploying a new PostgreSQL cluster in a modern AWS EC2 instance. You just attached a new 500GB EBS volume via the AWS console, and AWS told you it would be attached at `/dev/sdf`. 

You SSH into the instance and run `ls -l /dev/sdf`, but you get:
`ls: cannot access '/dev/sdf': No such file or directory`

You suspect that because it's a newer instance type, the kernel is using the NVMe interface, and `udev` is assigning a completely different name to the device. 

## Goal
1. What command or directory would you check to find all the block devices currently attached to the system, specifically looking for NVMe drives?
2. If you found a drive named `/dev/nvme1n1`, what `udevadm` command could you run to view all its kernel and udev attributes to confirm it's the 500GB drive you just attached?

## Solution
1. One way to do it is doing `ls -l /dev | grep "^b" | grep "nvme"`
2. `udevadm info --query=all --name=/dev/nvme1n1`. Not sure what attribute i should look for in there. 

**Correction:**
**Grade: 100% (Excellent)**

1. `ls -l /dev | grep "^b" | grep "nvme"` is a perfectly valid and clever way to do this using raw bash tools! Another simpler way if you just want to see disks is `lsblk` or checking `/sys/block/`.
2. `udevadm info --query=all --name=/dev/nvme1n1` is exactly the right command! As for what attribute you should look for, you'd typically look for `E: ID_SERIAL=` or `E: ID_SERIAL_SHORT=`. In AWS specifically, EBS volumes expose their volume ID (e.g., `vol-0abcd123456789`) in the serial number field, allowing you to definitively match the kernel device to the AWS console!
