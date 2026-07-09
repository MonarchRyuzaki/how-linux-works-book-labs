# Task 02: SSH Man-in-the-Middle Trap

## Scenario (Mode B: Production Outage)
Your company just upgraded the hardware for its jump-server (`localhost` for this exercise, usually `jump.corp.com`). When you attempt to SSH into it to fix a Sev-1 issue, SSH aggressively blocks your connection, screaming:
`WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!`

You know for a fact the infrastructure team just replaced the server's OS, meaning new host keys were generated. You need to bypass this error securely to stop the outage.

## Objective
1. Identify exactly which file and line number is storing the old, invalidated host key.
2. Surgically remove ONLY the offending key without deleting the rest of your trusted hosts.
3. Successfully SSH into `localhost`.

## Setup (Mode A: Break It For Me)
Run this command to deliberately poison your `known_hosts` file with a fake key for localhost:
```bash
# Corrupt the known_hosts file for localhost
ssh-keygen -R localhost 2>/dev/null
echo "localhost ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFAKEKEYBADBADBADBADBADBADBADBADBADBADBADBADBAD" >> ~/.ssh/known_hosts
echo "Host keys poisoned."
```

## Solution
*(Note: The simulation setup struggled to replicate locally due to IPv6 (`::1`) loopback fallback resolving the host cleanly. However, in a production environment, here is what would happen and how to solve it).*

1. **What was supposed to happen:** Running `ssh localhost` would violently reject the connection with a `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` error. This happens when the server is rebuilt (generating a new host key) but your client still has the old fingerprint saved in `~/.ssh/known_hosts`.
2. **How to fix it:** The massive error message explicitly tells you the file (`~/.ssh/known_hosts`) and line number of the bad key. You could manually open the file in `vim` and delete the line.
3. **The SRE way to fix it:** Run `ssh-keygen -R localhost`. This tool natively understands the file (including hashed hostnames) and surgically removes ONLY the keys for `localhost`, preserving the rest of your trusted infrastructure.
4. Finally, running `ssh localhost` again would prompt you to accept the brand new key, securely logging you in!
