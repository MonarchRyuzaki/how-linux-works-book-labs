# Task 04: The CI/CD Trojan Horse

**Mode:** Break It For Me (Mode A)

## Scenario
You are auditing the security of your company's Jenkins CI/CD pipeline. To allow the Jenkins worker container to build Docker images, the previous platform engineer configured it with a "Docker-out-of-Docker" (DooD) setup.

The Jenkins worker container was launched with this specific volume mount:
`-v /var/run/docker.sock:/var/run/docker.sock`

You tell the engineering director this is a massive security vulnerability. He pushes back: *"It's just a socket file so it can build images. The Jenkins worker is still locked securely inside a container. It can't touch the host machine's root filesystem."*

## Instructions
Your goal is to prove him wrong. Assume you have a bash shell *inside* the Jenkins worker container.
  Write the exact `docker` command you would run inside this container to spawn a new container that mounts the host machine's entire root filesystem (`/`) to `/host_root` inside the new container, effectively giving you unrestricted read/write access to the physical host's core files.

---

## Your Solution
1. `docker run -it -v /:/host_root alpine`

**Correction:**
Absolutely perfect! This is exactly the exploit.

Because `/var/run/docker.sock` is mounted into the Jenkins container, the `docker` command you execute inside the container isn't talking to an isolated daemon—it's talking directly to the physical host's Docker daemon. The host daemon blindly accepts the command and spawns a new container *on the host*, mounting the physical host's `/` directory to `/host_root`. You just bypassed the container isolation entirely and gained root access to the physical machine!
