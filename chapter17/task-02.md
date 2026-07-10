# Task 02: The Ghost Town of Overlay2

**Mode:** Production Outage Translation (Mode B)

## Scenario
A severe disk exhaustion alert triggers on one of your Kubernetes worker nodes. 

You SSH in and run `df -h`. The root filesystem `/` is at 100%. 
You run `sudo du -sh /var/lib/docker/*` and discover that `/var/lib/docker/overlay2/` is consuming 200GB of disk space.

You immediately check the 3 running containers using `docker ps`. They are stateless Nginx proxies. You `docker exec` into them and check their filesystems—they are only using a few megabytes each. 

*"If the running containers are tiny, why is the overlay2 directory taking up 200GB?"* asks the junior engineer.

## Instructions
1. Explain how Docker's lifecycle management and the overlay filesystem architecture can lead to this specific disk exhaustion scenario, even when `docker ps` only shows small, healthy containers.
2. Provide the exact Docker commands you would use to investigate and permanently clear this 200GB of bloat without affecting the 3 running Nginx proxies.

---

## Your Solution
Not sure explain this to me.

**Correction:**
No problem, this is a very common SRE trap!

**The Overlayfs Trap:** Docker uses an overlay filesystem. It takes a read-only image (like Alpine or Ubuntu) and places a writable layer ("upperdir") on top of it. Whenever a container writes a file or a log, it goes into this layer. 
When a container stops or crashes, **Docker does not delete this writable layer.** It preserves it in case you want to restart the container or inspect its remains. If you have an environment where containers are constantly crashing and restarting, or if you run cron jobs that spin up containers, you will accumulate thousands of "exited" containers. Their abandoned writable layers will slowly fill `/var/lib/docker/overlay2/` to 100%.

**The Fix:** 
1. Check for exited containers: `docker ps -a`
2. Permanently delete all stopped containers, dangling images, and unused networks to reclaim the space: `docker system prune -a` (or `docker container prune`).
