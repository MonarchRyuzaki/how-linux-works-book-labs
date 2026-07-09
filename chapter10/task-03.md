# Task 03: The Silent API (tcpdump primitives)

## Scenario (Mode B: Production Outage)
A new metrics agent is supposed to be pushing telemetry over UDP to port `8125` (StatsD). The backend monitoring dashboard is completely empty. The developer swears the agent is sending the packets, but the network team claims the firewall isn't blocking anything. 
As the SRE, you must prove whether the UDP packets are actually leaving the server's network interface.

## Objective
1. Construct a `tcpdump` command using primitives to capture ONLY UDP traffic destined for port 8125.
2. Watch the traffic. 
3. Prove that the developer's application is indeed transmitting payloads.

## Setup (Mode A: Break It For Me)
Run this command to simulate the developer's application sending metrics in the background:
```bash
# Sends a fake metric packet to UDP 8125 every 2 seconds
nohup bash -c "while true; do echo 'myapp.cpu.usage:42|g' > /dev/udp/127.0.0.1/8125; sleep 2; done" >/dev/null 2>&1 &
echo "Metrics agent simulator started."
```

## Solution
1. sudo tcpdump -i any udp port 8125
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
22:34:56.161811 lo    In  IP localhost.54492 > localhost.8125: UDP, length 21
22:34:58.164051 lo    In  IP localhost.44821 > localhost.8125: UDP, length 21
22:35:00.166383 lo    In  IP localhost.53042 > localhost.8125: UDP, length 21
22:35:02.168686 lo    In  IP localhost.38726 > localhost.8125: UDP, length 21
^C
4 packets captured
8 packets received by filter
0 packets dropped by kernel

### Correction:
**Pass!** Perfect execution. You correctly identified that listening on the wrong interface (`wlo1`) would drop local loopback traffic, and you successfully used `-i any` to catch the UDP packets on port 8125. This is a critical debugging skill when a developer claims their app is sending data but the dashboard is empty!
