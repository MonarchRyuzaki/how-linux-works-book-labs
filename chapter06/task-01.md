# Task 1: The Legacy App Migration

## Scenario (Mode B - Production Outage)
You've just joined a new team. They have a legacy Node.js application running on a critical production server. The current deployment process is literally: SSH into the box and run `nohup node /opt/app/server.js &`. 

Last night, the server rebooted for kernel patching. The app didn't come back up because no one was awake to manually start it. A Sev-2 outage ensued. 

Your manager has tasked you with bringing this service into the 21st century using systemd.

## Requirements
1. Write the exact contents of a `legacy-app.service` systemd unit file.
2. It should run the application as the `app-user` user, not as root.
3. It must start automatically on boot.
4. What commands will you run to install, start, and verify this new unit?

## Solution
1. 
File name = legacy-app.service
```
[Unit]
Descrption=A critical Node.js application
Requires=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/node /opt/app/server.js
ExecReload=/usr/bin/node /opt/app/server.js 
Restart=always
User=app-user
```

4. The commands are as follows
```bash
sudo systemctl daemon-reload
sudo systemctl enable legacy-app.service # To start on boot 
sudo systemctl start legacy-app.service # To start now 
sudo systemctl status legacy-app.service # To check if it is crashed
journalctl -u legacy-app.service # To verify
```

**Correction:**
Great start! A few SRE tips to make it bulletproof:
1. **The `[Install]` Section:** You missed `WantedBy=multi-user.target`. Without this, `systemctl enable` won't know *when* to start the app, and it won't actually create the symlink to start on boot!
2. **Typos:** `Descrption=` should be `Description=`. (systemd silently ignores typos, which is dangerous!).
3. **ExecReload:** You put `/usr/bin/node /opt/app/server.js` here. This would spawn a *second* Node process, which would likely crash because port 80/443 is already in use. It's best to omit `ExecReload` unless the app specifically handles something like `kill -HUP $MAINPID`.
4. **Ordering:** `Requires=network-online.target` is good, but you also need `After=network-online.target` to ensure it waits for the network to be ready before starting.
