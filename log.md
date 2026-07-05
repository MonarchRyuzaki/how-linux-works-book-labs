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
