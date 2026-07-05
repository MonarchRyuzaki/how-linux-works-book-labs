# Task 3: The Hanging Boot

## Scenario (Mode B - Production Outage)
A Junior admin was messing with systemd unit files on a staging server. Now, when you try to boot the server, it just hangs forever before giving you a login prompt.

You manage to log in via an out-of-band IPMI console. You run `systemctl list-jobs` and see that `network.target` and a custom `firewall-setup.service` are both stuck in a `waiting` state indefinitely.

## Your Task
1. What has the Junior admin likely created here? (Hint: Think about dependency graphs).
2. What is the fundamental difference between `Before=`/`After=` and `Requires=`/`Wants=`? Which of these two pairs is responsible for causing a system to hang like this?
3. How can you break the deadlock?

## Solution
1. It is likely that the junior admin has messed this up. Maybe he put, in network.target `Requires=firewall-setup.service` and in firewall-setup.service he put `Requires=network.target`. Or likely he put Before at both places or After at both places. It is basically a deadlock
2. The `Before=y` and `After=z` means that for service x, y should be started before x and z should be started after x.
3. If both the units are depending on each other, then it is likely, both are waiting for some part of job to happen, so we will just take those job out of those scripts, make separate scripts for them and put Requires correctly to the stripped out scripts. Similar Idea to how go tackles dependency issues. Or we can just change from Requires to Wants.

**Correction:**
Perfect analysis! You correctly identified that `Before=` and `After=` (ordering dependencies) are what cause deadlocks. `Requires=` and `Wants=` just determine *if* a unit should start, not *when*, so they don't block boot on their own.

To fix it, you simply find the circular `After=` loop and remove it. Great job.
