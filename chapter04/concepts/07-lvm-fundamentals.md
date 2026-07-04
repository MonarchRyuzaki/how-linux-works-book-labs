# 7. LVM Fundamentals

### Core Concept
LVM (Logical Volume Manager) allows SREs to resize partitions dynamically and span volumes across multiple physical hard drives. It works by chopping disks into tiny, uniform chunks called **Physical Extents (PE)**, usually 4MB in size. Resizing an LVM volume just means grabbing more loose PEs and stapling them to the end of the map.

### Essential Commands
```bash
# 1. PHYSICAL VOLUMES (PV): Take control of raw disks
sudo pvcreate /dev/sdb /dev/sdc

# 2. VOLUME GROUPS (VG): Create a bucket pooling all PEs together
sudo vgcreate my_pool /dev/sdb /dev/sdc

# 3. LOGICAL VOLUMES (LV): Scoop PEs out of the bucket to make a partition
sudo lvcreate -L 500M -n my_vol my_pool

# RESIZING (Extending)
# Add 200MB to the logical volume
sudo lvextend -L +200M /dev/my_pool/my_vol
# Crucial Step: Tell the filesystem to expand into the newly added LVM space
sudo resize2fs /dev/my_pool/my_vol

# Pro Tip: Do both of the above steps in a single command using the -r (--resizefs) flag!
sudo lvextend -r -L +200M /dev/my_pool/my_vol
```

### Caveats
Extending an LV (`lvextend`) is safe and can be done live while mounted. Shrinking an LV (`lvreduce`) is extremely dangerous. If you shrink the LV before shrinking the filesystem *inside* it, you will slice off the end of your data and corrupt the disk. Always backup before shrinking.
