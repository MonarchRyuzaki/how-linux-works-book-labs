# Task 2: The Silent Crash
**Concept:** `strace`, System Calls, `ENOENT`
**Difficulty:** Mid SRE

## The Scenario (Mode B)
A proprietary risk-calculation binary, `jpmc-risk-calc`, has just been deployed to a new UAT environment. When you try to start it, it immediately exits with exit code 1 and prints absolutely nothing to `stdout` or `/var/log/syslog`. 
Because it's a closed-source binary, you can't read the code to see why it's crashing. 

## The Setup (Mode A)
Run this to create the "proprietary binary":
```bash
cat << 'EOF' > /tmp/jpmc-risk-calc
#!/bin/bash
# Simulating a binary that silently fails if a config is missing
if [ ! -f /etc/jpmc/risk-engine.conf ]; then
    exit 1
fi
echo "Risk engine started successfully!"
EOF
chmod +x /tmp/jpmc-risk-calc
```

## The Mission
1. You are forbidden from reading the source code of `/tmp/jpmc-risk-calc` with `cat`. Treat it like a compiled ELF binary.
2. Use `strace` to execute the file and analyze the system calls.
3. Find the missing file (`ENOENT`) that is causing the application to panic and crash.
4. Fix the issue by creating the required file/directory so the program successfully prints "Risk engine started successfully!".

## Solution
strace /tmp/jpmc-risk-calc |& grep ENOENT
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "/etc/jpmc/risk-engine.conf", 0x7ffca61b8f30, 0) = -1 ENOENT (No such file or directory)

sudo mkdir /etc/jpmc

sudo touch /etc/jpmc/risk-engine.conf

strace /tmp/jpmc-risk-calc |& grep ENOENT
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)

/tmp/jpmc-risk-calc
Risk engine started successfully!


**Correction:**
Perfectly executed. Piping `strace` stderr to `grep ENOENT` is the absolute fastest way to find missing configuration files or dynamic libraries that cause silent crashes. This skill alone will save you hours of debugging proprietary binaries at JPMC.
