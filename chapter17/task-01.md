# Task 01: The Orphan Zombie Outbreak

**Mode:** Production Outage Translation (Mode B) & Break It For Me (Mode A)

## Scenario
You are the on-call SRE. PagerDuty goes off: **Host Node 04 is throwing "Resource temporarily unavailable" errors.**

You SSH into the host and notice that the PID count is maxed out. You track the issue down to a specific Docker container running a legacy multi-threaded data processor. 
When you run `docker top <container_id>`, you see hundreds of processes listed as `[Z] (defunct)` (Zombies). 

The developer insists: *"The container works fine! My application just spawns worker threads, does the job, and then the workers exit. Why are they turning into zombies and taking down the whole node?"*

## Instructions
1. Explain **architecturally** why a containerized application that spawns and kills child processes might leave them as zombies, even though this doesn't happen when running the exact same binary directly on the host machine.
2. What is the single Docker run flag you would add to instantly fix this architectural flaw without modifying the developer's code? 

---

## Your Solution
1. This is beacause, by default docker containers dont have the init process, which is supposed to take care of zombies and orphans. 
2. To fix this we need to run containers with --init flag. 

**Correction:**
Great job! You hit the nail on the head. 

**Why it happens:** In Linux, when a process dies, it becomes a "zombie" until its parent calls `wait()` to collect its exit code. If the parent dies first, the process becomes an "orphan" and gets adopted by PID 1 (the `init` process like systemd), which reaps it. However, Docker creates a brand new **PID Namespace** for the container, meaning your application becomes PID 1. Most applications (like Node.js or Python scripts) aren't programmed to act like an OS `init` system, so they simply ignore the dead child processes, letting zombies pile up until the host runs out of PIDs.
**The Fix:** You correctly identified the `--init` flag. This flag injects a tiny, dedicated init process (like `tini`) as PID 1, which correctly reaps all zombies, pushing your app to PID 2.
