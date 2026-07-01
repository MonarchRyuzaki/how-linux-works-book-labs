# Task 01: The Silently Failing Backup

## Scenario (Junior/Mid SRE Level)
You are on-call at JPMC. A nightly cron job is supposed to back up all application logs and write any errors to a file. The command running in the cron job is:
```bash
tar cvf /backup/logs.tar /var/log/app/*.log > /backup/backup-status.out
```

The application owners are complaining that the backups are missing, but when you check `/backup/backup-status.out`, the file is completely empty.

## The Challenge
1. **The Trap**: Why is the `.out` file empty even if the `tar` command is failing? What Unix concept explains this?
Solution: IO Redirection, Because we are using > instead of 2>&1 or 2>, the problem statement says that we are supposed to write errors to a file but we are writing the output that is why .out file is empty

2. **The Glob**: If there are *no* `.log` files in `/var/log/app/` tonight, what exactly does the bash shell pass to the `tar` command? How does this differ from Windows?
Solution: I am not sure how Windows handle it but I think, if no .log files are there, the shell Globbing falls back to using * as a name itself, meaning it searches for a file named "*.log"

3. **The Fix**: Rewrite the command so that **both** standard output (the files being backed up) and standard error (the error messages) are captured safely in `/backup/backup-status.out`.
Solution: 
```bash
tar cvf /backup/logs.tar /var/log/app/*.log > /backup/backup-status.out 2>&1
```

I am not sure if using only 2>&1 instead of > will work, but 2>&1 at the end should work.

**Correction:**
1. Spot on! `>` only redirects `stdout` (File Descriptor 1). Errors go to `stderr` (File Descriptor 2), which is why they bypassed your redirect.
2. Correct! Bash passes the literal string `*.log` to `tar` if no files match. 
3. `> /backup/backup-status.out 2>&1` is perfect. It points `stdout` to the file, and then points `stderr` (`2`) to where `stdout` (`1`) is pointing. (Also note it's `/backup/logs.tar` not `/backups/`).

## Related Concepts (Chapter 2)
- Shell Globbing Expansion
- Standard Output vs Standard Error
- I/O Redirection (`>` vs `2>&1`)
