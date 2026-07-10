# Task 02: Ghost Files and the Trap Escape (Temporary Files & Signal Handling)

**Scenario:**
A legacy log rotation script downloads massive daily log files to `/tmp`, parses them with `awk`, and then deletes them. Because the parsing takes a long time, Junior engineers often get impatient and hit `CTRL+C` to abort the script. 
Over the last month, the `/tmp` directory has completely filled up with orphaned log files because the `rm` command at the end of the script is never reached when the script is interrupted.

Here is the basic structure of the script:
```bash
#!/bin/sh
TMPFILE=$(mktemp /tmp/log_parse.XXXXXX)
echo "Downloading logs to $TMPFILE..."
# Simulating a long running process
sleep 15
echo "Parsing complete."
rm -f $TMPFILE
```

**Your Objective:**
1. Copy this script to your workspace and run it. Press `CTRL+C` during the sleep. Verify that the temp file is left behind in `/tmp`.
2. Fix the script by implementing a `trap` that guarantees the temporary file is deleted even if the user aborts the script.

Provide your solution below:
### Solution
1. We insert the line trap "rm -f $TMPFILE", so that the tmp files are removed if user aborts the script.

**Correction:**
**Grade: 50% (Conceptually Correct, Syntax incomplete)**

You correctly identified that we need to use `trap "rm -f $TMPFILE"`. However, the `trap` command needs to specify *which* signals it should catch. `CTRL+C` generates the `INT` (Interrupt) signal. 

Also, if the trap catches the signal, the shell will just continue running the rest of the script unless we explicitly tell it to `exit`.

**Correct Fixed Script:**
```bash
#!/bin/sh
TMPFILE=$(mktemp /tmp/log_parse.XXXXXX)

# Trap the INT signal and exit 1 after cleaning up
trap "rm -f $TMPFILE; exit 1" INT

echo "Downloading logs to $TMPFILE..."
sleep 15
echo "Parsing complete."
rm -f $TMPFILE
```
