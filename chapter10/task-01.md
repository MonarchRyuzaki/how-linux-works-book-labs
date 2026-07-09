# Task 01: The Mysterious Listening Port

## Scenario (Mode B: Production Outage)
An alert just fired: A legacy internal application is failing to start because it claims its required port (TCP 8080) is "Already in use". You check the process manager, but the legacy app is definitely dead. There is a rogue, undocumented process silently hogging the port on your server.

## Objective
1. Identify the exact PID and name of the rogue process holding port 8080 open.
2. Without killing the process yet, figure out what application-layer text it is expecting. (Hint: use a raw TCP client to connect to it and say "hello").
3. Terminate the rogue process and prove the port is freed.

## Setup (Mode A: Break It For Me)
Run this block in your terminal to simulate the rogue process holding the port:
```bash
# Simulates a rogue application holding the port and waiting for input
cat << 'EOF' > /tmp/rogue_server.py
import socketserver
class RogueHandler(socketserver.StreamRequestHandler):
    def handle(self):
        self.wfile.write(b"Rogue Service v1.0. Send command:\n")
        data = self.rfile.readline().strip()
        self.wfile.write(b"Unknown command: " + data + b"\n")
socketserver.TCPServer(("", 8080), RogueHandler).serve_forever()
EOF
nohup python3 /tmp/rogue_server.py >/dev/null 2>&1 &
echo "Rogue process deployed."
```

## Solution
1.  ss -tlnp | grep 8080
LISTEN 0      5            0.0.0.0:8080       0.0.0.0:*    users:(("python3",pid=62207,fd=3)) 
PID = 62207, Looks like it is a python process.
2. nc localhost 8080
Rogue Service v1.0. Send command:
hello
Unknown command: hello
^C
3.  kill 62207
[1]+  Terminated              nohup python3 /tmp/rogue_server.py > /dev/null 2>&1
ss -tlnp | grep 8080 gives no output

### Correction:
**Pass!** Excellent troubleshooting. You successfully identified the process holding the port using `ss -tlnp`, verified its application-layer behavior by directly connecting to it using a raw TCP socket (`nc`), and cleanly terminated it.
