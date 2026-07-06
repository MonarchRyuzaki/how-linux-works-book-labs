# Task 04: The Silent Cron Failure

## Scenario (Mode B: Production Outage)
A developer added a critical backup script to the root user's crontab:
`0 2 * * * /opt/backup/run_backup.sh`

The developer tested the script manually (`sudo /opt/backup/run_backup.sh`) and it worked perfectly. However, when cron runs it at 2:00 AM, it fails silently. The script relies on the `$DB_PASSWORD` environment variable, which is defined in `/root/.bashrc`.

## Objective
1. Why does the script work when run manually by the developer, but fail when executed by cron? (Hint: Think about the environment).
2. The tech lead wants you to migrate this cron job to a **Systemd Timer Unit** for better logging and reliability. Write the basic structure for the `.timer` unit and the `.service` unit to replace this cron job. 

## Solution
1. When we run it manually in shell, shell by default runs .bashrc on startup so env variable is already loaded. When it is not run via shell, the env variable is not set so the script fails to execute.
2. .timer unit
```
[Unit]
Description=Timer for run backup service.

[Timer]
OnCalendar=0 2 * * *
```

.service unit
```
[Unit]
Description=Backup service

[Service]
Type=simple
ExecStart=/opt/backup/run_backup.sh
Restart=on-failure
Environment="DB_PASSWORD=database-password"
```

**Correction:**
* **Grade: 9/10!** Excellent diagnosis.
* **1:** Spot on! Cron executes scripts in a non-interactive, non-login shell environment. It does not source `~/.bashrc` or `~/.profile`, which is the #1 cause of cron failures.
* **2:** Great job separating the `.timer` and `.service` logic and injecting the `Environment=`! Just one syntax catch: systemd's `OnCalendar` doesn't use the standard 5-star cron format. Instead of `0 2 * * *`, systemd expects `*-*-* 02:00:00` (Year-Month-Day Hour:Min:Sec).
