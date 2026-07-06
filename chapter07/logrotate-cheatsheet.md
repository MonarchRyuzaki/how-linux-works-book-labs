# The SRE logrotate Configuration Cheat Sheet

## 📂 File Locations
- **`/etc/logrotate.conf`**: The global configuration file. Usually contains default settings and an `include` directive for the `logrotate.d` directory.
- **`/etc/logrotate.d/`**: Where you (and package managers) drop application-specific configuration files (e.g., `/etc/logrotate.d/nginx`).

---

## 🛠️ Core Configuration Syntax
*A standard logrotate block targets a specific log file or pattern.*

```text
/var/log/my-app/*.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
```

---

## ⏱️ Rotation Triggers
*When does a file actually get rotated?*

| Directive | What it does |
| :--- | :--- |
| `daily` / `weekly` / `monthly` | Rotates based on the calendar (e.g., once a day). |
| `size 50M` | Rotates **only** if the file exceeds 50 Megabytes, regardless of the time schedule. |
| `maxsize 100M` | Rotates on the time schedule, BUT will also rotate early if the file exceeds 100M. |
| `rotate 7` | **CRITICAL:** Keeps 7 old logs before deleting the oldest one (e.g., `.1` to `.7`). If set to `0`, old logs are deleted immediately without being saved! |

---

## 📦 Compression and Behavior
*How should the files be stored and handled?*

| Directive | What it does |
| :--- | :--- |
| `compress` | Compresses old log files (usually with `gzip`, producing `.gz` files). |
| `delaycompress` | Waits until the *next* rotation cycle to compress the file. (Useful if the application might still be writing to the recently rotated `.1` file). |
| `missingok` | If the log file doesn't exist, don't throw an error—just quietly move on. |
| `notifempty` | If the log file is currently 0 bytes (empty), don't bother rotating it. |

---

## 🔄 The Open File Descriptor Problem (Zero Downtime)
*When logrotate renames `app.log` to `app.log.1`, the application will continue writing to `app.log.1` because the kernel tracks the inode, not the filename! How do you fix this without restarting the app?*

| Directive / Technique | What it does |
| :--- | :--- |
| `copytruncate` | **The Brute Force Fix:** Instead of moving the file, it copies the data to `app.log.1`, and then `truncate -s 0` the original file. The app never loses its file descriptor. *(Risk: You might lose a few bytes written during the split-second copy phase).* |
| `create 0640 www-data www-data` | **The Standard Fix:** Moves `app.log` to `app.log.1`, then instantly creates a brand new, empty `app.log` with these exact permissions and ownership. |
| `postrotate` ... `endscript` | **The Reload Fix:** Used alongside `create`. After creating the new empty file, runs a bash command to tell the application to reload its file descriptors (e.g., `systemctl reload nginx` or sending a `SIGHUP` to the PID). |

### Example using `postrotate`:
```text
/var/log/nginx/*.log {
    daily
    rotate 14
    create 0640 nginx adm
    postrotate
        [ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`
    endscript
}
```

---

## ⚡ The "I Just Wrote a Config" Workflow
Whenever you create or modify a file in `/etc/logrotate.d/`, test it immediately!

```bash
# Dry Run (Test without actually touching files, parses config for syntax errors)
sudo logrotate -d /etc/logrotate.d/my-app

# Force Rotation (Run it right now, ignoring time/size checks)
sudo logrotate -f /etc/logrotate.d/my-app

# Check logrotate's internal state (When did it last run?)
cat /var/lib/logrotate/status
```
