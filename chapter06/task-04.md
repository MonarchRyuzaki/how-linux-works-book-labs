# Task 4: The "active (exited)" Trap

## Scenario (Mode B - Production Outage)
You need to run a quick bash script (`/usr/local/bin/setup-iptables.sh`) once on boot to apply some temporary firewall rules. You write a systemd service for it:

```ini
[Unit]
Description=Firewall Setup

[Service]
Type=simple
ExecStart=/usr/local/bin/setup-iptables.sh

[Install]
WantedBy=multi-user.target
```

You boot the server. The script runs perfectly and applies the rules. But when you run `systemctl status firewall-setup`, systemd reports the service as `inactive (dead)`. Because it thinks the service died, other units that depend on the firewall might not start, or systemd might try to restart it if you had a Restart directive.

## Your Task
1. Why does systemd consider this service "dead" after the script finishes? (Hint: What does `Type=simple` assume about the process?)
2. How should you modify the `[Service]` section (specifically the `Type` and another directive) so that systemd considers the service `active (exited)` and successfully running even after the bash script completes?

## Solution
1. Its Because we are using Service Type = Simple, when the execution is complete, it will report as dead or inactive, better to use type=oneshot and RemainAfterExit=yes 
2. Type=oneshot, RemainAfterExit=yes

**Correction:**
Flawless! `Type=oneshot` paired with `RemainAfterExit=yes` is the exact textbook solution for one-off setup scripts like iptables rules.
