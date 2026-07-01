# Task 04: The Rogue Background Process

## Scenario (Junior/Mid SRE Level)
You log into a shared jump host to perform some maintenance. Suddenly, random text starts polluting your terminal window every 5 seconds, making it impossible to type commands clearly. 

Someone clearly left a script running in the background (`&`) that is writing to the terminal.

## The Challenge
1. **The Search**: What `ps` command (using the BSD style flags mentioned in the chapter) combined with `grep` can you use to list all running processes across all users to identify the rogue script?
Solution:
```bash
ps aux 
```
2. **The Graceful Stop**: Once you find the PID (let's say it's 4022), you want to terminate it gracefully to let it clean up its temporary files. What command do you use? (Hint: What is the default signal?).
Solution:
Solution: I think we can use SIGINT or SIGTERM for graceful shutdown
3. **The Brutal Stop**: The script ignores your first attempt. It's stuck in an uninterruptible loop. What is the absolute last-resort command to force the kernel to instantly execute it?
Solution: kill -9 or kill SIGKILL

**Correction:**
1. `ps aux` is perfect.
2. `SIGTERM` (which is the default when you just run `kill <PID>`) is exactly right. It politely asks the process to shut down and clean up.
3. `kill -9` (or `kill -KILL`) is correct! This bypasses the application completely and tells the kernel to instantly destroy the process and reclaim its memory.

## Related Concepts (Chapter 2)
- Background Processes (`&`)
- Job Control and Signals
- `ps`, `kill`, and `grep`
