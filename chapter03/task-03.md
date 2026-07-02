# Task 03: The Missing Symlink Mystery

## Scenario (Mode B: Production Outage)
A legacy backup script on your server is hardcoded to write database dumps directly to a block device at `/dev/backup-disk`. 

Usually, this works perfectly because a custom `udev` rule detects the backup drive based on its unique serial number and automatically creates a symlink from `/dev/sdb` (or whatever letter it gets) to `/dev/backup-disk`.

However, the server rebooted, the disk got assigned to `/dev/sdc`, and the `/dev/backup-disk` symlink was never created. Backups are failing.

## Goal
1. In which directory would you look to find and fix the custom `udev` rule?
2. How would you use `udevadm` to manually find the `ID_SERIAL` attribute of `/dev/sdc` so you can verify if the rule's serial number matches the physical disk?

## Solution
1. The directory to look for is `/etc/udev/rules.d`
2. We can use `udevadm info --query=all --name=/dev/sdc | grep -i "serial"`

**Correction:**
**Grade: 100% (Great SRE Intuition)**

1. Yes! Custom rules live in `/etc/udev/rules.d/`. (The default system rules live in `/lib/udev/rules.d/`, but you shouldn't edit those because OS updates will overwrite them). 
2. Piping the `udevadm` output to `grep` is a great practical move to quickly filter for the serial number (like `ID_SERIAL`). Once you have that, you can update the rule in `/etc/udev/rules.d/` and then run `udevadm trigger` to apply the fix without needing to reboot again!
