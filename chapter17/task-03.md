# Task 03: The Silent Bridge

**Mode:** Production Outage Translation (Mode B)

## Scenario
A team is migrating a legacy monolithic web application to Docker. 
They proudly show you their deployment:
```bash
docker run -d --name legacy_web legacy_image:v1
```

They tell you the app listens on port `8080` internally. However, when the Load Balancer tries to reach the host machine on port `8080`, the connection is immediately refused. 

The developer runs `docker exec legacy_web curl localhost:8080` and proves the app is responding *inside* the container. 

*"The app is running! Why is the host machine refusing the connection from the outside?"* they ask.

## Instructions
1. Based on your knowledge of Docker's default networking (network namespaces and `docker0`), explain exactly why traffic reaching the physical host's `eth0` is not reaching the container's virtual interface.
2. What specific mechanism (involving `iptables` and NAT) does Docker use to bridge this gap, and what is the exact `docker run` flag the developer missed to configure it?

---

## Your Solution
I dont know the exact answer but its because, there is no bridge network so the host machine cant access the docker container because there is no docker bridge network to reach inside the docker container.

**Correction:**
You have the right intuition (it is a networking gap), but the mechanics are slightly different!

**Why it happens:** There *is* actually a bridge network created by Docker (`docker0`), but the container lives inside an isolated **Network Namespace**. Its `eth0` interface has a private IP (e.g., `172.17.0.2`). The host's physical network interface (`eth0`) knows absolutely nothing about this private IP or what's running inside the namespace. If outside traffic hits the host on port 8080, the host's kernel just drops it because nothing in the host's main namespace is listening on 8080.

**The Fix:** Docker bridges this gap using **iptables NAT (Network Address Translation)**. To tell Docker to create the NAT rule that forwards host traffic to the container namespace, the developer missed the port mapping flag:
`-p 8080:8080`
