# Task 02: The Rogue Sync (rsync --delete)

**Scenario:**
You need to make the remote backup directory `/mnt/backup/web` an *exact replica* of the local `/var/www/html/` directory. The backup directory currently has 500GB of old files that are no longer needed, and you want them gone.

You draft the following command:
```bash
rsync -a --delete /var/www/html/ backupserver:/mnt/backup/web/
```

However, a senior engineer stops you. "Never run `--delete` on production without a dry run. If you messed up the source path, you could wipe the entire 500GB backup instantly."

**Your Objective:**
Modify your command to perform a "dry run" along with verbose output. This will print exactly which files would be deleted, without actually deleting anything.

### Solution
1. Add a -nv flag. 

**Correction:**
**Grade: 100%**
Correct. `rsync -anv --delete /var/www/html/ backupserver:/mnt/backup/web/`. Always use `-n` (dry run) and `-v` (verbose) to see exactly what `--delete` is going to destroy before you run it for real.
