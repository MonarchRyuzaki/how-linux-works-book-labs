# Task 02: The Locked Out Web Server

## Scenario (Junior/Mid SRE Level)
A junior developer deployed a new static landing page for an internal JPMC tool. However, anyone who visits the site gets an HTTP **403 Forbidden** error.

You ssh into the web server. The Nginx process is running as the user `www-data`.
You run `ls -l /var/www/html/index.html` and see:
```text
-rw------- 1 root root 1024 Jul 1 12:00 /var/www/html/index.html
```

## The Challenge
1. **The UGO Hierarchy**: Based on the exact permissions shown, break down exactly how the Linux kernel evaluates the `www-data` user's request to read this file. Which buckets (User, Group, Other) are checked, and why is access denied?
Solution: The current permissions allow the user named "root" to read and write the file, the group named "root" and the others cant read, write or execute the file.
The access is denied because the nginx is running as user "www-data", which is not root hence does not have the permission to see the file

2. **The Absolute Fix**: Provide the exact `chmod` command (using absolute numbers, e.g., 755) that would fix this without changing the file's ownership, allowing the web server to read it but not write to it.
Solution: 
```bash
sudo chmod 604 /var/www/html/index.html
```


3. **The Ownership Fix**: Alternatively, provide the `chown` command to change the owner of the file to `www-data`. 
Solution:
```bash
sudo chown www-data:www-data /var/www/html/index.html 
```

**Correction:**
1. Perfect explanation of the UGO check order! The kernel sees `www-data` is not `root`, so it falls to the "Other" bucket, which has `---` (no access).
2. `chmod 604` works! It gives `rw-` to the owner, `---` to the group, and `r--` (4) to others. (A more common standard is `644` so the group can also read, but 604 strictly satisfies the prompt).
3. `sudo chown www-data:www-data` is absolutely correct.

## Related Concepts (Chapter 2)
- File Modes and Permissions (RWX)
- `chmod` and `chown`
- Process execution contexts 
