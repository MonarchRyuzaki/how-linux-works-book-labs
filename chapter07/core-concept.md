# Chapter 7: System Configuration: Logging, System Time, Batch Jobs, and Users

## Structural Outline
* 7.1 System Logging
  * 7.1.1 Checking Your Log Setup
  * 7.1.2 Searching and Monitoring Logs
  * 7.1.3 Logfile Rotation
  * 7.1.4 Journal Maintenance
* 7.2 Configuration Files
* 7.3 User Management
  * 7.3.1 Users and /etc/passwd
  * 7.3.2 Groups and /etc/group
  * 7.3.3 Shadow Passwords
* 7.4 getty and login
* 7.5 System Time and Network Time Protocol
* 7.6 Scheduling Tasks with Cron
* 7.7 Systemd Timer Units
* 7.8 Timer Units Running as Regular Users
* 7.9 User Identification and Privileges
  * 7.9.1 UID and EUID
  * 7.9.2 The setuid Mechanism
  * 7.9.3 User Identification, Authentication, and Authorization
  * 7.9.4 Using Libraries for User Information
* 7.10 Pluggable Authentication Modules (PAM)
  * 7.10.1 PAM Configuration
  * 7.10.2 Tips on PAM Configuration Syntax
  * 7.10.3 PAM and Passwords

## Core Concepts
* **System Logging (`journald` & `syslog`)**: Modern Linux relies on `journald` to collect logs across the OS natively via systemd. It captures `stdout/stderr` of services and stores them in binary format. `logrotate` handles plain-text syslog file rotations to prevent disk exhaustion.
* **User Management & Passwords**: The `/etc/passwd` file maps usernames to UIDs, but no longer stores passwords for security reasons. Instead, `/etc/shadow` securely holds encrypted hashes. This structure is foundational for permissions across the OS.
* **Time Syncing**: Maintaining accurate system time is critical for logs and cryptography. `systemd-timesyncd` or `ntpd` keeps the OS synced with global NTP servers, distinct from the motherboard's hardware clock.
* **Scheduled Jobs (`cron` vs `systemd timers`)**: Cron is the legacy, simple way to run scheduled scripts. Systemd Timer units (`.timer` + `.service`) are the modern standard, offering vastly superior logging, resource limits, and dependency management for scheduled jobs.
* **PAM (Pluggable Authentication Modules)**: A central framework configured in `/etc/pam.d/` that handles all authentication (SSH, sudo, su). It uses a "stack" of dynamic rules to dictate password policies, MFA enforcement, and account lockouts without recompiling applications.

## Commands & Utilities
* `journalctl`: Primary tool for querying systemd journal logs. (e.g., `journalctl -u nginx` for a service, `-S -4h` for time).
* `logrotate`: Utility that rotates, compresses, and purges old log files based on `/etc/logrotate.conf`.
* `timedatectl`: Used to view and manage system time, timezones, and NTP synchronization.
* `crontab`: Legacy utility to manage user-specific recurring schedules (`crontab -e`).
* `systemctl list-timers`: Shows active systemd timers, their next execution time, and their corresponding service units.
* `getent`: Queries system databases like `/etc/passwd` or `/etc/group` even if they are handled over a network (like LDAP/AD).
* `/etc/pam.d/`: Directory containing application-specific PAM configurations for authentication stacking.
