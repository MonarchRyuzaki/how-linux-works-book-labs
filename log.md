# Activity Log

*(For instructions, user profile, and AI workflow, see [WORKFLOW.md](WORKFLOW.md))*

### [2026-07-01] Chapter 2: Basic Commands and Directory Hierarchy
- **Action**: Used `pdf_chapter_extractor` to process pages 35-70.
- **Outcome**: Generated `chapter02/core-concept.md` containing the structural outline and core concepts.
- **Action**: Used `active_practice_generator` to create 7 structured scenario-based tasks (`task-01.md` through `task-07.md`).
- **Action**: Graded user solutions and appended `**Correction:**` blocks to all task files.
- **Status**: Chapter 2 completed. Ready for Chapter 3.

### [2026-07-02] Chapter 3: Devices
- **Action**: Used `pdf_chapter_extractor` to process pages 71-91 and generated `chapter03/core-concept.md`.
- **Action**: Used `active_practice_generator` to create 6 scenario-based tasks (`task-01.md` through `task-06.md`).
- **Action**: Graded user solutions and appended `**Correction:**` blocks to all task files.
- **Status**: Chapter 3 completed. Ready for Chapter 4.

### [2026-07-02] Chapter 4: Disks and Filesystems
- **Action**: Extracted pages 93-139 to `chapter04/raw_text.txt`.
- **Action**: User paused standard SOP to review theory up to page 97. Applied `deep-reader` skill principles to break down partitions, MBR vs GPT, and fdisk vs parted.
- **Action**: User completed reading the chapter. Practiced LVM and partitioning via loop devices safely.
- **Action**: Extracted 10 key concepts into separate markdown files in `chapter04/concepts/` (Partitioning, Filesystems, Mount/UUID, Remounting, fsck, Swap, LVM, Device Mapper, Inodes/Hardlinks, Bitmaps).
- **Action**: Used `active_practice_generator` to create 7 structured scenario-based tasks (`task-01.md` through `task-07.md`), including "DO" lab tasks for LVM and loop devices.
- **Action**: Graded user solutions and appended `**Correction:**` blocks to all task files.
- **Status**: Chapter 4 completed. Ready for Chapter 5.

### [2026-07-05] Chapter 5: How the Linux Kernel Boots
- **Action**: Extracted pages 141-159 to `chapter05/raw_text.txt`.
- **Action**: By user request, skipped task generation due to low SRE relevance (GRUB/UEFI/Chainloading). Generated a streamlined `chapter05/core-concept.md` focused exclusively on `dmesg`, kernel initialization, and boot parameters.
- **Status**: Chapter 5 completed. Ready for Chapter 6.

### [2026-07-05] Chapter 6: How User Space Starts (init & systemd)
- **Action**: Extracted pages 161-189 to `chapter06/raw_text.txt`.
- **Action**: User requested a pivot in learning style. Updated `WORKFLOW.md` to prioritize hands-on tasks over reading theory.
- **Action**: Generated `chapter06/core-concept.md` as a quick reference.
- **Action**: Used `active_practice_generator` to create 7 scenario-based tasks (`task-01.md` through `task-07.md`) blending Mode B and Mode A.
- **Action**: Graded user solutions and appended `**Correction:**` blocks to all task files.
- **Status**: Chapter 6 completed. Ready for Chapter 7.

### [2026-07-06] Chapter 7: System Configuration: Logging, System Time, Batch Jobs, and Users
- **Action**: Extracted pages 191-222 to `chapter07/raw_text.txt`.
- **Action**: Generated `chapter07/core-concept.md` containing the structural outline and core concepts.
- **Action**: Used `active_practice_generator` to create 6 scenario-based tasks (`task-01.md` through `task-06.md`).
- **Action**: Graded user solutions and appended `**Correction:**` blocks to all task files.
- **Status**: Chapter 7 completed. Ready for Chapter 8.

### [2026-07-06] Chapter 8: A Closer Look at Processes and Resource Utilization
- **Action**: Extracted pages 223-248 to `chapter08/raw_text.txt`.
- **Action**: Generated `chapter08/core-concept.md` containing the structural outline and core concepts.
- **Action**: Used `active_practice_generator` to create 6 scenario-based tasks (`task-01.md` through `task-06.md`).
- **Action**: Graded user solutions and appended `**Correction:**` blocks to all task files.
- **Status**: Chapter 8 completed. Ready for Chapter 9.

### [2026-07-08] Chapter 9: Understanding Your Network and Its Configuration
- **Action**: Extracted pages 247-292 to `chapter09/raw_text.txt`.
- **Action**: Generated `chapter09/core-concept.md` containing the structural outline and core concepts.
- **Action**: User paused standard SOP to map bare-metal networking concepts to AWS equivalents (VPC, IGW, NAT, Security Groups).
- **Action**: Extracted key SRE concepts into `chapter09/concepts/` (Unicast, DHCP/SLAAC, Linux Router/NAT, Firewalls/iptables, TCP/UDP Ports, DNS Resolution Pipeline).
- **Action**: Generated `chapter09/active-practice.md` featuring a Mode E "AWS VPC on Bare-Metal Linux" masterclass lab using `ip netns`.
- **Action**: Generated `chapter09/dns-tasks.md` featuring Mode B and Mode A DNS sabotage tasks.
- **Status**: Chapter 9 completed. Ready for Chapter 10.

### [2026-07-09] Chapter 10: Network Applications and Services
- **Action**: Extracted pages 293-314 to `chapter10/raw_text.txt`.
- **Action**: Generated `chapter10/core-concept.md` containing the structural outline and core concepts, later appending a Deep Dive on Unix Domain Sockets.
- **Action**: Used `active_practice_generator` to create 6 scenario-based tasks (`task-01.md` through `task-06.md`).
- **Action**: Debugged tricky SSH fallback mechanisms (`::1` loopback resolving) with the user during Task 02.
- **Action**: Graded user solutions and appended `**Correction:**` blocks to all task files.
- **Status**: Chapter 10 completed. Ready for Chapter 11.
