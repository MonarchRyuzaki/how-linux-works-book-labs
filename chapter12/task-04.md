# Task 04: The Open Door (Python SimpleHTTPServer)

**Scenario:**
A developer comes to you asking for a quick way to transfer a 50MB log file from a restricted testing server to their local machine. They don't have SSH keys set up for `scp`, but they can `curl` the server's IP address.

You tell them to `cd` into the log directory and run `python -m SimpleHTTPServer`.
They do it from `/` (the root directory) by accident instead of the log directory.

**Your Objective:**
1. What port does this server bind to by default?
2. What is the massive security risk of running this command from the `/` directory on a network that isn't perfectly isolated?

### Solution
1. By default its port 8000
2. running the server from / means, you are exposing everything in your machine.

**Correction:**
**Grade: 100%**
Exactly. It binds to `0.0.0.0:8000` by default, meaning anyone on the network can hit it. If you run it from `/`, they can navigate to `http://IP:8000/etc/shadow` or `http://IP:8000/home/user/.ssh/id_rsa` and download your keys. Very dangerous!
