# Task 06: The Broken Config Update (Symlinks)

## Scenario (Junior/Mid SRE Level)
You need to update a configuration file for an application at `/etc/app/config.yml`.

When you run `ls -l /etc/app/config.yml`, you see:
```text
lrwxrwxrwx 1 root root 22 Jul 1 09:00 /etc/app/config.yml -> /opt/app/default.yml
```

You open `/etc/app/config.yml` in `vi` and try to save your changes, but `vi` gives you an error: "E212: Can't open file for writing".

## The Challenge
1. **The Deception**: Why does the `ls -l` output show `rwxrwxrwx` (world-writable) for the symlink, but `vi` still denies you permission to write? What permissions does the kernel actually check?
Solution: My hypothesis is, the symlink is world-writable but looks like the actual file /opt/app/default.yml is not world-writable
2. **The Danger**: If you use `rm /etc/app/config.yml` to delete the symlink, what happens to the target file `/opt/app/default.yml`?
Solution: It can happen that the actual file in app directory is deleted.
3. **The Fix**: The original `default.yml` is owned by `root` and is read-only. What is the safest way to provide a custom configuration for this app without altering the global default? (Explain how you would use `cp` and `ln -s`).
Solution: 
```bash
cp /opt/app/default.yml ./config.yml
ln -s ./config.yml /etc/app/config.yml

```

**Correction:**
1. Spot on! The `ls -l` permissions of a symlink (`lrwxrwxrwx`) are ignored. The kernel always follows the link and checks the permissions of the *target* file.
2. **Incorrect**: `rm` on a symlink *only deletes the link itself*. It does absolutely nothing to the target file! This is a common misconception, but the target file `/opt/app/default.yml` remains perfectly safe.
3. Your logic is right, but three small tweaks: First, `rm /etc/app/config.yml` to remove the old link. Second, copy it to a permanent location (e.g., `cp /opt/app/default.yml /etc/app/custom_config.yml`). Third, link that permanent file: `ln -s /etc/app/custom_config.yml /etc/app/config.yml`.
## Related Concepts (Chapter 2)
- Symbolic Links (`ln -s`)
- File Permissions on Links
- Directory Navigation and Copying
