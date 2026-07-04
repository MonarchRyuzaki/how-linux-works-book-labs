# 4. Remounting and Permissions

### Core Concept
Remounting allows you to change the properties (like read/write permissions) of an actively mounted filesystem *without* having to fully unmount it first. 

### Usecase for SREs
If a Linux filesystem detects internal corruption or a hardware failure, it will automatically remount itself in **Read-Only (`ro`) mode** to protect the data from further damage. Once you fix the underlying issue, you can flip the filesystem back to Read-Write without taking the system offline.

### Essential Commands
```bash
# Force a mounted filesystem into Read-Only mode (safe)
sudo mount -o remount,ro /data

# Flip a filesystem back to Read-Write (after fixing errors)
sudo mount -o remount,rw /data

# If a drive refuses to unmount because it is "busy", find the culprit process
sudo lsof +D /data
```

### Caveat
You cannot fully unmount (`umount`) a drive if a user or process has a file open inside it ("Device is busy"). However, you *can* usually issue a remount on a busy drive, making it a critical tool for live server maintenance.
