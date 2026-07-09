# Active Practice: DNS Troubleshooting (Mode B & A)

The VPC lab covers routing and firewalls perfectly, but DNS is often the true culprit of an SRE outage. Here are two isolated, high-stakes DNS tasks to complete.

## Task 1: The Staging Database Trap (Mode B - Outage)

**The Scenario:** 
You are on-call. A developer franticly messages you: *"Our backend service is crashing! It's trying to connect to the production database at `db.prod.internal`, but it keeps getting 'Connection Refused'!"*

You verify the production database is healthy and the AWS Security Groups are completely open. You run a quick test from your machine and it resolves to `10.0.5.50`.

**The Break Script:**
Run this command on your machine to simulate the broken state of the backend server:
```bash
echo "127.0.0.1 db.prod.internal" | sudo tee -a /etc/hosts > /dev/null
```

**Your Mission:**
1. Try to ping `db.prod.internal`. Notice how it behaves.
2. Run `host db.prod.internal`. Notice how it returns something completely different (or fails) compared to the ping!
3. Using your knowledge of the Linux DNS Resolution Pipeline (Section 9.15), explain *why* `ping` and `host` are disagreeing.
4. Find the rogue configuration and fix the server so it connects to the real database again.

Solution:
1. ping db.prod.internal
PING db.prod.internal (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.033 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.056 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.061 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.063 ms
64 bytes from localhost (127.0.0.1): icmp_seq=5 ttl=64 time=0.055 ms
^C
--- db.prod.internal ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4073ms
rtt min/avg/max/mdev = 0.033/0.053/0.063/0.010 ms

2. host db.prod.internal
db.prod.internal has address 127.0.0.1
Host db.prod.internal not found: 3(NXDOMAIN)

3. It looks like, since the DNS resolution starts with looking at `/etc/nsswitch.conf`, when `db.prod.internal` is to be resolved, it sees the hosts line is at files then dns, so it will first see `/etc/hosts`, from there it will be resolved to `127.0.0.0.1`, instead of using dns to resolve to `10.0.5.50`. 

4. To fix it, we just need to remove the `127.0.0.1 db.prod.internal` from /etc/hosts. 
---

## Task 2: The Silent Hijack (Mode A - Break It)

**The Scenario:**
A rogue script ran on a server and slightly modified a core configuration file. Suddenly, a bunch of internal microservices that talk to `localhost` have started failing randomly, but external internet access (`curl google.com`) works perfectly fine.

**The Break Script:**
Run this command to sabotage your system's DNS pipeline:
```bash
sudo sed -i 's/hosts:.*files.*dns.*/hosts: dns files/' /etc/nsswitch.conf
```

**Your Mission:**
1. Try to ping an external site (`ping google.com`). It works.
2. Try to ping a made-up local hostname that you add to `/etc/hosts`. It will fail!
3. Using the concepts from Section 9.15.4, figure out what the sabotage script actually did to the DNS pipeline. 
4. Why is `/etc/hosts` suddenly being ignored? 
5. Fix the configuration file and restore the original pipeline order.

Solution: 
1. Yeah it works
2. Yeah I pinged amazon, but it pings localhost instead
3. But from the script it looks like, the order for seeing hosts is reversed from `files then dns` to `dns then files`
4. Yeah fixed the reverse order

**Corrections:**
- **Task 1:** Spot on diagnosis! You correctly identified that `ping` obeys `/etc/nsswitch.conf` and checks the `/etc/hosts` file first. The crucial SRE takeaway here is that standard networking applications (like `curl`, `ping`, and python/node apps) obey `nsswitch.conf`, but pure DNS diagnostic tools (like `host`, `dig`, and `nslookup`) are hardcoded to **bypass** `nsswitch.conf` and query the DNS server directly. This is exactly why `ping` and `host` can return conflicting answers during an outage, causing massive confusion unless you know the pipeline!
- **Task 2:** Perfect. By flipping the order to `hosts: dns files`, the system queries external DNS *before* checking local overrides. This breaks internal development routes and causes unnecessary network delays if the DNS server is slow. Restoring it to `hosts: files dns` is the correct fix.

