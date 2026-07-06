# Task 05: The Locked Castle (Mode A: Break It For Me)

## Scenario
You are investigating a server where a Junior Admin accidentally broke the Pluggable Authentication Modules (PAM) configuration for the `su` command. 

Here is the current `/etc/pam.d/su` configuration:
```text
auth       sufficient   pam_rootok.so
auth       required     pam_deny.so
auth       required     pam_unix.so
```

Currently, no one (not even root!) can use `su` to switch users. Every attempt returns "Authentication failure".

## Objective
1. Trace the PAM execution flow. Why does this exact configuration guarantee failure for any normal user typing a correct password? 
2. Fix the configuration so that `root` can `su` without a password, but normal users are prompted for a password and authenticated via `pam_unix.so`.

## Solution
1. If the user is root, authentication should succed, if it is not root, then goes to next rule, it is pam_deny means, it will deny for all other users.
2. We will switch the order, first pam_unix then pam_deny. 

**Correction:**
* **Grade: 10/10!** 
* **1:** Exactly right! Because `pam_rootok.so` is `sufficient`, root skips the rest of the file. Normal users fail `pam_rootok` and hit `pam_deny` which guarantees their authentication fails.
* **2:** Perfect solution. Changing the order to `pam_unix` first allows normal users to type their password. If they get it right, they are authenticated. `pam_deny` then safely catches anyone else who somehow bypassed `pam_unix`.
