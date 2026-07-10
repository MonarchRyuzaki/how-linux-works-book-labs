# Task 06: Storage Architecture Interrogation

**Scenario:**
You are proposing a shared storage solution for a cluster of 50 Linux rendering nodes. The rendering application needs extremely high-throughput, low-latency, random access to massive 3D asset files.

A junior engineer proposes using **SSHFS** for all 50 nodes to connect back to a central file server because "it's encrypted and easy to set up."

**Your Objective:**
Shoot down the junior engineer's proposal using concepts from Chapter 12. Explain why SSHFS is a terrible choice for high-throughput, latency-sensitive random access, and what traditional alternative might be better for an isolated, secure rendering cluster.

### Solution
1. We cant have high-throughput cuz there is overhead of encryption, translation and transport.
2. NFS might be better idk.

**Correction:**
**Grade: 100%**
100% correct, and your "idk" is actually spot on. NFS (Network File System) is the traditional, high-performance, low-overhead way to share storage across a high-speed LAN for things like rendering clusters. SSHFS encryption overhead would utterly bottleneck 50 nodes trying to load 3D assets simultaneously.
