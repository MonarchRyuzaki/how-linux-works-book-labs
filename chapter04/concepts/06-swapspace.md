# 6. Swap Space

### Core Concept
Swap space is a designated partition (or file) on the hard drive that acts as "overflow" or "virtual" RAM. If the server runs out of physical RAM, the Linux kernel moves idle memory pages from RAM into Swap to prevent the system from crashing (Out Of Memory / OOM killer).

### Essential Commands
```bash
# Format a partition specifically for swap
sudo mkswap /dev/sda2

# Activate the swap space instantly
sudo swapon /dev/sda2

# Deactivate the swap space (moves everything back to RAM if space permits)
sudo swapoff /dev/sda2

# View current RAM and Swap usage
free -h
```

### Caveats
Hard drives (even NVMe SSDs) are thousands of times slower than physical RAM. If your server is actively relying on Swap (a state called "thrashing"), CPU usage will spike to 100% waiting for the disk, and the server will become totally unresponsive. Swap is a safety net, not a replacement for RAM.
