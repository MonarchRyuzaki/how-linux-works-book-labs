# Chapter 12: Network File Transfer and Sharing

## 1. Quick Copy (Python HTTP Server)
- `python -m SimpleHTTPServer`: Starts a basic web server on port 8000, serving the current directory.
- **Security:** Completely unauthenticated and unencrypted. Only for trusted local networks.

## 2. rsync (File Synchronization)
- Replicates file/directory structures efficiently by only transferring differences.
- **Core Flags:**
  - `-a` (Archive): Preserves permissions, symlinks, times, etc.
  - `-n` (Dry Run): Simulates the transfer. Always use with `--delete` to preview destruction.
  - `-v` (Verbose): Shows files being transferred.
  - `--delete`: Exact replica mode. Deletes files on destination that are not in the source.
  - `-z`: Compresses data in transit (good for slow WAN links, bad for fast LANs due to CPU overhead).
  - `--bwlimit=KBPS`: Limits bandwidth to prevent saturating uplinks.
- **The Trailing Slash Trap:** 
  - `rsync -a dir host:dest` creates `dest/dir`.
  - `rsync -a dir/ host:dest` copies the *contents* of `dir` directly into `dest`.

## 3. Samba and CIFS (Windows Interoperability)
- **Samba Server:** Configured via `smb.conf`. Daemons: `smbd` and `nmbd`.
- **Authentication:** Independent TDB password database managed via `smbpasswd`.
- **Clients:** 
  - `smbclient`: FTP-like interface to browse and interact with remote shares.
  - **CIFS Mount:** `mount -t cifs SERVER:sharename mountpoint -o user=X,pass=Y`

## 4. SSHFS (Linux-to-Linux User-Space Sharing)
- Mounts a remote directory over an SSH connection using FUSE.
- **Command:** `sshfs user@host:/dir /local/mountpoint`
- **Unmount:** `fusermount -u /local/mountpoint` (User-space unmount).
- **Pros/Cons:** Zero server setup (only needs SFTP enabled on SSH server), secure, but suffers from encryption/transport performance overhead.

## 5. NFS (Network File System)
- Traditional Unix file sharing. High performance, but natively low security (host-based access, cleartext).
- **Command:** `mount -t nfs server:/dir /local/mountpoint`

## 6. FUSE & Cloud Storage
- FUSE (File System in User Space) allows modern clients (like S3 buckets) to be mounted as local directories without kernel modifications, bridging modern cloud APIs with traditional POSIX file access.

---

## Summary Comparison

| Method | Features | Advantages | Disadvantages | Best Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Python HTTP Server** | Ad-hoc web server on port 8000 | Zero config, works instantly | Unencrypted, unauthenticated, read-only | Quick, temporary LAN file sharing |
| **`rsync`** | Differential sync, compression, dry-runs | Efficiently syncs large/deep directories, bandwidth limits | Not a real-time mount, syntax is prone to human error (`/`) | Automated backups, massive data migrations |
| **Samba / CIFS** | SMB protocol interoperability | Bridges Linux with Windows networks | Complex server config (`smb.conf`), separate password DB | Sharing files/printers in mixed Windows/Linux environments |
| **SSHFS** | User-space (FUSE) mount via SSH | Zero server setup, natively encrypted | High CPU/overhead, hangs when network drops | Remote development, mounting codebases securely |
| **NFS** | Traditional kernel-level Unix mount | Low overhead, extremely fast throughput | Weak native security (IP-based, cleartext) | High-performance isolated LAN clusters (e.g., rendering nodes) |
| **Cloud FUSE (S3)**| User-space translation of cloud APIs | Infinite scalability, no hardware management | Very high latency compared to local blocks | Archival, managed off-site backups |
