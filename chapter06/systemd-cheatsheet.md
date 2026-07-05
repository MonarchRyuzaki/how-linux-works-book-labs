# The SRE systemd Unit File Cheat Sheet

## 📂 File Locations
- **`/etc/systemd/system/`**: Where you (the admin) create and modify your unit files. (Changes here override package defaults).
- **`/lib/systemd/system/`**: Where `apt`/`yum` install default unit files. Do not edit these directly.

---

## 🏗️ Core Unit Types
- **`.service`**: Runs a daemon or script (e.g., `nginx.service`).
- **`.target`**: A logical grouping of units, used as synchronization points (e.g., `multi-user.target` is roughly equivalent to Runlevel 3).
- **`.socket`**: Listens on a port/socket and spawns a `.service` on-demand when traffic arrives.
- **`.timer`**: A modern replacement for cron jobs; triggers `.service` units on a schedule.

---

## 🛠️ The `[Unit]` Section
*Controls metadata, dependencies, and execution ordering.*

| Key | Values / Usage | What it does |
| :--- | :--- | :--- |
| `Description=` | Text string | Human-readable name shown in `systemctl status`. |
| `Requires=` | Space-separated unit names | **Strict Dependency:** If this unit starts, the listed units MUST start. If a required unit crashes/stops, systemd forcibly stops THIS unit too. |
| `Wants=` | Space-separated unit names | **Weak Dependency:** If this unit starts, systemd *tries* to start the listed units. If they fail, this unit keeps running. (Highly recommended over `Requires`). |
| `Before=` | Space-separated unit names | **Ordering:** Ensures THIS unit fully starts *before* the listed units start. |
| `After=` | Space-separated unit names | **Ordering:** Ensures THIS unit waits to start until *after* the listed units have started. |
| `Conflicts=` | Space-separated unit names | **Anti-dependency:** If this unit starts, systemd immediately stops the listed units (and vice versa). |

---

## ⚙️ The `[Service]` Section
*Controls how the process is executed, monitored, and restarted.*

| Key | Values / Usage | What it does |
| :--- | :--- | :--- |
| `Type=` | `simple`, `forking`, `oneshot`, `notify` | **`simple`**: (Default). The process runs in the foreground. systemd considers it "started" immediately.<br>**`forking`**: The process forks to the background (legacy daemons).<br>**`oneshot`**: For scripts that run and exit. Use with `RemainAfterExit=yes` so systemd considers it "active" after finishing.<br>**`notify`**: Like `simple`, but the daemon explicitly sends a signal via sd_notify() to systemd when it's ready. |
| `ExecStart=` | Absolute path to binary/script | The exact command to start the service. (e.g., `/usr/bin/node /opt/app.js`). |
| `ExecStop=` | Absolute path to binary/script | Command to run when stopping. (Optional: systemd normally just sends a SIGTERM if omitted). |
| `ExecReload=` | Absolute path to binary/script | Command to gracefully reload config without killing the process (e.g., `kill -HUP $MAINPID`). |
| `Restart=` | `no`, `always`, `on-failure` | **`always`**: Restart if it crashes or exits cleanly.<br>**`on-failure`**: Restart only if it crashes (non-zero exit code or segfault). |
| `RestartSec=` | Time in seconds (e.g., `5s`) | How long systemd waits before attempting to restart a crashed service. |
| `StartLimitBurst=` | Integer (e.g., `3`) | How many times the service is allowed to restart within `StartLimitIntervalSec`. |
| `StartLimitIntervalSec=`| Time (e.g., `10`) | The time window for crash-loop limits. (e.g., "Give up if it crashes 3 times in 10 seconds"). |
| `User=` / `Group=` | Linux username/group | **CRITICAL:** Runs the service as this specific user instead of root. |
| `WorkingDirectory=`| Absolute path | Changes the directory (`cd`) before running `ExecStart`. |
| `Environment=` | `VAR=value` | Injects environment variables into the process. |

---

## 🔌 The `[Socket]` Section (For .socket files)
*Controls on-demand network listening.*

| Key | Values / Usage | What it does |
| :--- | :--- | :--- |
| `ListenStream=` | Port number (e.g., `8080`) | Listens on a TCP port. |
| `ListenDatagram=`| Port number (e.g., `53`) | Listens on a UDP port. |
| `Accept=` | `true` or `false` | **`false`** (Default): Spawns ONE service daemon and passes the socket to it.<br>**`true`**: Spawns a NEW instance of the service for *every single incoming connection* (like classic `inetd`). |

---

## 🚀 The `[Install]` Section
*Controls what happens when you run `systemctl enable`.*

| Key | Values / Usage | What it does |
| :--- | :--- | :--- |
| `WantedBy=` | Target name (e.g., `multi-user.target`) | When you `enable` this unit, systemd creates a symlink so that this unit starts automatically when `multi-user.target` is reached (which happens on normal system boot). |
| `RequiredBy=` | Target name | Same as `WantedBy`, but creates a strict requirement. |

---

## ⚡ The "I Just Edited a File" Workflow
Whenever you create or modify a file in `/etc/systemd/system/`, you MUST run:
```bash
sudo systemctl daemon-reload
```
Then, you can manage it:
```bash
sudo systemctl enable my-app.service   # Set to autostart on boot
sudo systemctl start my-app.service    # Start it right now
systemctl status my-app.service        # Check if it crashed
journalctl -u my-app.service           # Read the logs if it crashed
```
