# Task 6: Socket Activation (On-Demand)

## Scenario (Mode B - Production Outage)
We have a massive internal `admin-dashboard` tool that eats up 2GB of RAM. The problem is, it's only used by the Lead SRE once a week. The rest of the time, it's just wasting resources.

Your manager asks you to configure it so that the service doesn't run at all on boot. Instead, systemd should listen on TCP port `8080`. The moment a web browser tries to hit port `8080`, systemd should instantly spawn the `admin-dashboard` service to handle the request.

## Your Task
1. Explain how systemd achieves this resource-parallelized startup (what two types of unit files do you need?).
2. Assuming the service is named `admin-dashboard.service`, write out the exact contents of the `.socket` file needed to make this work on port 8080.
3. Which unit do you `systemctl enable` and `systemctl start`? The service unit or the socket unit?

## Solution
1. We will be using two unit files, admin-dashboard.service and admin-dashboard.socket, we will just start the socket and not the service, so whenever the socket is hit, it starts the service and the resources are saved since service is not permanently running. A better thing to do would be to add a second service which closes the admin-dashboard.service since after starting the service we need to stop it too to prevent it from eating the RAM.
2. File Name = admin-dashboard.socket
```
[Unit]
Description=Socket for admin-dashboard
Requires=networking.target
Before=admin-dashboard.socket

[Socket]
ListenStream=8080 # Listen to port 8080 for starting the service

[Install]
RequiredBy=admin-dashboard.service
```
3. We will enable and start the socket unit not the service.

**Correction:**
Excellent! You correctly identified that we only start/enable the `.socket` unit. 

A few small tweaks for production:
1. In `[Unit]`, `Before=admin-dashboard.socket` is unnecessary (systemd handles socket->service ordering automatically).
2. In `[Install]`, a socket file usually uses `WantedBy=sockets.target`, not `RequiredBy=admin-dashboard.service`. 
But your conceptual understanding of how socket activation works is 100% correct!
