# Task 02: The Rogue Administrator

## Scenario (Mode B: Production Outage)
An alert fires: an unknown script is executing with root privileges. You check `/etc/passwd` and `/etc/shadow` and notice something deeply suspicious. There is a new user named `service_account`, but its UID (User ID) is set to `0`. Furthermore, its password hash field in `/etc/shadow` has been manually edited to `*`.

## Objective
1. Why is setting the UID to `0` for `service_account` dangerous, even if the username isn't `root`? How does the kernel view this user?
2. What does the `*` in the password hash field of `/etc/shadow` actually do? Can someone log in via SSH with this account using a password?

## Solution
1. Setting UID to 0 means, it is viewed as a administrator, means it has full access to everything
2. * in hash field, means that this account cant be logged in, So this means this account is not usable for SSH log in. 

**Correction:**
* **Grade: 10/10!**
* **1:** Exactly. The kernel doesn't care about the string `root`. It only checks if `UID == 0`. So `service_account` is practically a root backdoor.
* **2:** Correct. The `*` (or `!`) is not a valid cryptographic hash, so the PAM `pam_unix.so` module will always fail to verify any password typed in. (Note: They could still theoretically SSH in using SSH keys if `~/.ssh/authorized_keys` was set up, since SSH key authentication bypasses the shadow password check, but they can NEVER use a password!).
