# Task 03: The Uplink Killer (rsync Bandwidth)

**Scenario:**
You are transferring a massive 2TB database dump from your local datacenter to an offsite disaster recovery facility. 

You run:
```bash
rsync -a /data/db_dump.sql dr-site:/data/
```

Within 5 minutes, you get a call from the network team. The office internet connection is completely saturated. No one can make VoIP calls, and customer API requests are timing out. Your `rsync` upload is consuming 100% of the 1Gbps uplink.

**Your Objective:**
Rewrite the `rsync` command to limit the bandwidth to roughly 100 Mbps (100,000 Kbps) so the network team stops yelling at you. Also, add the flag to compress the data in transit to speed up the logical transfer.

### Solution
1. `rsync --bwlimit=100000 -az /data/db_dump.sql dr-site:/data/` 

**Correction:**
**Grade: 100%**
Perfect. `-z` adds compression to speed up the transfer of the compressible `.sql` text file, and `--bwlimit=100000` throttles the network bandwidth so you don't take down the company's uplink.
