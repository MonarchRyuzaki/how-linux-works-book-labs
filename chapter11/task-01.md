# Task 01: The Silently Failing Backup (Exit Codes & Subshells)

**Scenario:**
An automated database backup script runs every night on a crontab. It exports data, compresses it, and then uploads it to an S3 bucket. Yesterday, the production database disk filled up entirely (ENOSPC). The `pg_dump` command failed completely.
However, the cron job reported success, and the S3 bucket contains a 0-byte `.gz` file!

Here is the faulty script:
```bash
#!/bin/sh
# Generate the dump inside a subshell to avoid changing working dir
(cd /mnt/db_backups && pg_dump main_db > backup.sql)
gzip /mnt/db_backups/backup.sql
upload_to_s3 /mnt/db_backups/backup.sql.gz
```

**Your Objective:**
1. Why did the script continue and report a successful run despite `pg_dump` failing? Explain how Exit Codes (`$?`) and subshells interact here.
2. Rewrite the script to safely catch the error and abort immediately (exit code 1) if `pg_dump` fails. *Hint: Consider `|| exit 1`, `set -e`, or manually checking `$?`.*

Provide your solution below:
### Solution

**Correction:**
**(Not Attempted - Here is the solution)**

**1. Explanation:**
The script successfully finished because the commands inside `(...)` run in a **subshell**. The `pg_dump` failed (returning a non-zero exit code), but the subshell simply closed and the script moved on to the next line (`gzip`). `gzip` successfully compressed an empty file (resulting in a 0-byte `.gz` file), and `upload_to_s3` successfully uploaded that empty file. Because the final command succeeded, the script exited with a `0` (success), causing the cron job to think everything went perfectly.

**2. Fixed Script:**
To fix this, we need to explicitly check the exit code of the commands, or use `set -e` so the script aborts on the first failure.
```bash
#!/bin/sh
# Use set -e to abort immediately if any command fails
set -e

(cd /mnt/db_backups && pg_dump main_db > backup.sql) || exit 1
gzip /mnt/db_backups/backup.sql
upload_to_s3 /mnt/db_backups/backup.sql.gz
```
