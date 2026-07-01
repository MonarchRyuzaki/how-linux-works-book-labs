# Task 05: The "Command Not Found" Trap

## Scenario (Junior/Mid SRE Level)
A developer is trying to install a custom binary in their home directory and added a line to their `.bashrc` to update their path. They log out and log back in. 

Suddenly, nothing works. When they type `ls`, the shell says `bash: ls: command not found`. When they type `cat`, they get `bash: cat: command not found`. They are panicking.

## The Challenge
1. **The Root Cause**: Explain exactly what the developer likely typed in their `.bashrc` that completely destroyed their ability to run basic commands.
Solution: The user likely did export PATH="xyzpath" and not PATH="xyzpath:$PATH"
2. **The Recovery**: You need to fix their `.bashrc`. But since `vi` and `nano` also return "command not found", how can you execute the editor to fix the file? (Hint: Think about where core binaries live in the FHS).
Solution: We need to manually navigate to the binary, like /usr/bin/nano
3. **The Correct Syntax**: What is the correct way to prepend `/home/user/custom/bin` to the `PATH` environment variable without destroying the existing path?
Solution: PATH="/home/user/custom/bin:$PATH"

**Correction:**
1. Right! By not appending `:$PATH`, they wiped out `/bin` and `/usr/bin` from their path, so the shell couldn't find basic commands anymore.
2. Excellent! Core binaries are typically in `/bin` or `/usr/bin`, so running `/usr/bin/nano ~/.bashrc` explicitly tells the shell exactly where to look.
3. Your syntax `PATH="/home/user/custom/bin:$PATH"` is correct for a shell variable, but you need the `export` keyword so it gets passed to child processes! The complete fix is: `export PATH="/home/user/custom/bin:$PATH"`.

## Related Concepts (Chapter 2)
- Environment Variables (`export`)
- The Command Path (`PATH`)
- FHS (Filesystem Hierarchy Standard: `/bin`, `/usr/bin`)
