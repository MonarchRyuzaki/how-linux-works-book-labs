# Task 7: DO IT LIVE (Break It For Me)

## Scenario (Mode A - Break It)
Let's get our hands dirty on the terminal and build a self-healing systemd service.

First, create a highly unstable script. Run this in your terminal to create `/usr/local/bin/flakey.sh`:
```bash
sudo bash -c 'cat << EOF > /usr/local/bin/flakey.sh
#!/bin/bash
echo "Starting flakey service..."
sleep 2
echo "Oh no, a segmentation fault!"
exit 1
EOF'
sudo chmod +x /usr/local/bin/flakey.sh
```

## Your Task (Do this in your terminal)
1. Write a `flakey.service` unit file in `/etc/systemd/system/` that executes this script.
2. Configure the unit so that systemd **automatically restarts** the service if it crashes.
3. Configure it so that if the service restarts more than **3 times within a 10-second window**, systemd gives up and marks it as failed (to prevent infinite restart loops that burn CPU).
4. Run the service. Monitor it using `watch -n 1 systemctl status flakey.service` or check the logs with `journalctl -u flakey.service`.

Once you have it working, paste your `flakey.service` file below, along with the `journalctl` logs proving that it restarted and eventually gave up.

## Solution
1. 
```
[Unit]
Description="A flakey service"

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/bin/flakey.sh
StartLimitBurst=3
Restart=on-failure
StartLimitIntervalSec=10
```

2. I did that with daemon reload and starting the service
3. Present in the unit file 
4.
```
Jul 05 18:27:15 systemd[1]: Starting "A flakey service"...
Jul 05 18:27:15 flakey.sh[22708]: Starting flakey service...
Jul 05 18:27:17 flakey.sh[22708]: Oh no, a segmentation fault!
Jul 05 18:27:17 systemd[1]: flakey.service: Main process exited, code=exited, status=1/FAILURE
Jul 05 18:27:17 systemd[1]: flakey.service: Failed with result 'exit-code'.
Jul 05 18:27:17 systemd[1]: Failed to start "A flakey service".
Jul 05 18:27:17 systemd[1]: flakey.service: Scheduled restart job, restart counter is at 1.
Jul 05 18:27:17 systemd[1]: Stopped "A flakey service".
Jul 05 18:27:17 systemd[1]: Starting "A flakey service"...
Jul 05 18:27:17 flakey.sh[22746]: Starting flakey service...
Jul 05 18:27:19 flakey.sh[22746]: Oh no, a segmentation fault!
Jul 05 18:27:19 systemd[1]: flakey.service: Main process exited, code=exited, status=1/FAILURE
Jul 05 18:27:19 systemd[1]: flakey.service: Failed with result 'exit-code'.
Jul 05 18:27:19 systemd[1]: Failed to start "A flakey service".
Jul 05 18:27:19 systemd[1]: flakey.service: Scheduled restart job, restart counter is at 2.
Jul 05 18:27:19 systemd[1]: Stopped "A flakey service".
Jul 05 18:27:20 systemd[1]: Starting "A flakey service"...
Jul 05 18:27:20 flakey.sh[22750]: Starting flakey service...
Jul 05 18:27:22 flakey.sh[22750]: Oh no, a segmentation fault!
Jul 05 18:27:22 systemd[1]: flakey.service: Main process exited, code=exited, status=1/FAILURE
Jul 05 18:27:22 systemd[1]: flakey.service: Failed with result 'exit-code'.
Jul 05 18:27:22 systemd[1]: Failed to start "A flakey service".
Jul 05 18:27:22 systemd[1]: flakey.service: Scheduled restart job, restart counter is at 3.
Jul 05 18:27:22 systemd[1]: Stopped "A flakey service".
Jul 05 18:27:22 systemd[1]: flakey.service: Start request repeated too quickly.
Jul 05 18:27:22 systemd[1]: flakey.service: Failed with result 'exit-code'.
Jul 05 18:27:22 systemd[1]: Failed to start "A flakey service".
```

**Correction:**
Amazing job doing it live! The logs prove your setup worked perfectly. 
You correctly implemented `StartLimitBurst=3` and `StartLimitIntervalSec=10`, and the log output (`Start request repeated too quickly`) is exactly what we wanted to see.

*Minor note:* You used `Type=oneshot`. Usually, `oneshot` implies the script is expected to exit, so you don't typically use `Restart=on-failure` with it. `Type=simple` would be the more idiomatic choice for a daemon that is supposed to stay running. However, because your script exited with a failure code (`exit 1`), `Restart=on-failure` still triggered! 

Great hands-on troubleshooting.
