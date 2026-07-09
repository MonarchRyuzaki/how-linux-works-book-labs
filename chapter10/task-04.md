# Task 04: The Phantom 500 Error

## Scenario (Mode B: Production Outage)
A load balancer is reporting that a local web service is unhealthy. However, a simple `ping` works, and `netstat` shows the process is listening. You suspect the application layer is failing (e.g., returning an HTTP error), but the developer's automated scripts are hiding the raw headers. 

You need to speak HTTP manually to prove to the developer exactly what the server is returning.

## Objective
1. Connect to the service using a raw TCP client (like `telnet` or `nc`).
2. Manually type the raw HTTP/1.1 `GET` request headers to request the root `/` path.
3. Capture the exact HTTP Status Code the server returns. 
4. Alternatively, use `curl --trace-ascii` to generate a trace file and locate the HTTP response headers.

## Setup (Mode A: Break It For Me)
Run this command to deploy a buggy Python web server on port 8000 that intentionally throws a 500 Internal Server Error for every request:
```bash
cat << 'EOF' > /tmp/buggy_server.py
import http.server, socketserver
class BuggyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_error(500, "Internal Application Crash: NullPointer")
socketserver.TCPServer(("", 8000), BuggyHandler).serve_forever()
EOF
nohup python3 /tmp/buggy_server.py >/dev/null 2>&1 &
echo "Buggy web server deployed on port 8000."
```

## Solution
1. printf "GET / HTTP/1.1\r\nHost: example\r\nConnection: close\r\n\r\n" | nc localhost 8000
HTTP/1.0 500 Internal Application Crash: NullPointer
Server: BaseHTTP/0.6 Python/3.10.12
Date: Thu, 09 Jul 2026 17:08:51 GMT
Connection: close
Content-Type: text/html;charset=utf-8
Content-Length: 476

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
        "http://www.w3.org/TR/html4/strict.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
        <title>Error response</title>
    </head>
    <body>
        <h1>Error response</h1>
        <p>Error code: 500</p>
        <p>Message: Internal Application Crash: NullPointer.</p>
        <p>Error code explanation: 500 - Server got itself in trouble.</p>
    </body>
</html>

2. bat trace_file
───────┬──────────────────────────────────────────────────────────────────────────────────────
       │ File: trace_file
───────┼──────────────────────────────────────────────────────────────────────────────────────
   1   │ == Info:   Trying 127.0.0.1:8000...
   2   │ == Info: Connected to localhost (127.0.0.1) port 8000 (#0)
   3   │ => Send header, 78 bytes (0x4e)
   4   │ 0000: GET / HTTP/1.1
   5   │ 0010: Host: localhost:8000
   6   │ 0026: User-Agent: curl/7.81.0
   7   │ 003f: Accept: */*
   8   │ 004c:
   9   │ == Info: Mark bundle as not supporting multiuse
  10   │ == Info: HTTP 1.0, assume close after body
  11   │ <= Recv header, 54 bytes (0x36)
  12   │ 0000: HTTP/1.0 500 Internal Application Crash: NullPointer
  13   │ <= Recv header, 37 bytes (0x25)
  14   │ 0000: Server: BaseHTTP/0.6 Python/3.10.12
  15   │ <= Recv header, 37 bytes (0x25)
  16   │ 0000: Date: Thu, 09 Jul 2026 17:09:29 GMT
  17   │ <= Recv header, 19 bytes (0x13)
  18   │ 0000: Connection: close
  19   │ <= Recv header, 39 bytes (0x27)
  20   │ 0000: Content-Type: text/html;charset=utf-8
  21   │ <= Recv header, 21 bytes (0x15)
  22   │ 0000: Content-Length: 476
  23   │ <= Recv header, 2 bytes (0x2)
  24   │ 0000:
  25   │ <= Recv data, 476 bytes (0x1dc)
  26   │ 0000: <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN".        "http:
  27   │ 0040: //www.w3.org/TR/html4/strict.dtd">.<html>.    <head>.        <me
  28   │ 0080: ta http-equiv="Content-Type" content="text/html;charset=utf-8">.
  29   │ 00c0:         <title>Error response</title>.    </head>.    <body>.
  30   │ 0100:      <h1>Error response</h1>.        <p>Error code: 500</p>.
  31   │ 0140:     <p>Message: Internal Application Crash: NullPointer.</p>.
  32   │ 0180:      <p>Error code explanation: 500 - Server got itself in troub
  33   │ 01c0: le.</p>.    </body>.</html>.
  34   │ == Info: Closing connection 0

### Correction:
**Pass!** Fantastic work! You successfully bypassed the application layer using a raw TCP pipe (`nc`) to speak HTTP manually. You also beautifully demonstrated the `curl --trace-ascii` method, which is the gold standard for proving to a developer exactly what headers their app is sending over the wire before the browser or tooling hides it.
