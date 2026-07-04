# Task 01: The Phantom Drive

**Mode:** Production Outage Translation (Mode B)
**Calibrated Difficulty:** Junior/Mid SRE
**Target Concepts:** Mount/UUID, `/etc/fstab`, Boot Process

## Scenario
You are paged at 3:00 AM. A critical application server (`app-node-04`) was just rebooted for a routine kernel patch. However, it hasn't come back online. 

When you connect via the hypervisor console, you see the server is stuck in **Emergency Mode**. The system logs show the following error:
`Timed out waiting for device dev-sdb1.device.`
`Dependency failed for /data/app-db.`

You run `lsblk` and notice that the system currently has two drives: `/dev/sda` (the OS drive) and `/dev/sdc` (a 500GB data drive). There is no `/dev/sdb`. 

A senior engineer tells you: "Oh, we added a new SAN storage LUN to that host yesterday before the reboot. The hardware discovery order probably shifted."

## Your Task
1. Explain exactly why the server crashed into Emergency Mode based on the senior engineer's comment. What specific file is misconfigured?
2. How do you fix this permanently so that adding or removing drives in the future never causes this issue again? State the exact commands and configuration changes you would make.

---
### Your Solution
1. It is likely that they were using /dev/sdb for mounting the drive. The misconfigured file is /etc/fstab . They should have used UUID for mounting
2. They should first unmount the `/dev/sdc` by using `sudo umount /dev/sdc`, then they should get the UUID by running `sudo blkid` then mount it by `sudo mount UUID={The uuid from prev statement} /data/app-db`

**Correction:**
Great logic! You completely nailed the core issue: relying on hardware device names (`/dev/sdb1`) in `/etc/fstab` is dangerous because adding a new SAN LUN shifted the letters around. 

The only minor tweak in your solution: Since the server is stuck in "Emergency Mode", the drive actually *failed* to mount during boot, so you don't even need to unmount it! You would just run `blkid` to get the UUID, edit `/etc/fstab` to replace `/dev/sdb1` with `UUID=...`, and then type `exit` to let the server finish booting. Excellent troubleshooting!
