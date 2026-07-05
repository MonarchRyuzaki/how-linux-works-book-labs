# How Linux Works (JPMC SRE Prep) - Workflow & Instructions

## Overview
This repository contains notes and active practice labs for studying "How Linux Works", specifically tailored for JPMC Junior/Mid SRE interview prep and onboarding.

## User Profile & Calibrations
- **User Profile**: Fresher SRE starting at JPMC.
- **Current Baseline**: Has a highly practical cheatsheet covering advanced Linux/Systems concepts. Previous scenarios were heavily AI-assisted. **Crucially, learns best by DOING (hands-on) rather than reading dense theory.**
- **Pedagogical Goal**: Calibrate scenarios to a **Junior/Mid SRE level**. Build independent intuition. 
- **Pivot Strategy:** Treat the book as a reference, not a primary teacher. Output very concise `core-concept.md` summaries and immediately jump into `active_practice_generator` tasks. The struggle of solving tasks is the primary learning vehicle.

## Standard Operating Procedure (SOP)
When processing a new chapter in a fresh chat, the AI agent should read this file and follow this exact sequence:

1. **Extraction**: Use the `pdf_chapter_extractor` skill to extract the specified pages from the book.
2. **Summarization**: Parse the raw text and generate a `core-concept.md` file inside the corresponding chapter folder (e.g., `chapter03/core-concept.md`). Include the structural outline, core concepts, and a list of commands.
3. **Task Generation**: When the user requests practice, use the `active_practice_generator` skill (blending Mode B and Mode A) to create 6-7 realistic scenarios combining multiple concepts.
   - Save each task as a separate file named `task-01.md`, `task-02.md`, etc., inside the chapter folder.
4. **Grading & Correction**: Once the user completes the tasks and requests a review, evaluate their solutions.
   - For each task, append a section named `**Correction:**` directly beneath the user's `Solution` in the markdown file.
   - Provide encouraging, detailed feedback explaining why they are correct or where they went wrong.
5. **Logging**: After each major step (extraction, task generation, or chapter completion), append a brief summary to `log.md` in the root directory to preserve chronological continuity across chats.
