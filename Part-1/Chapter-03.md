# CHAPTER 3: FILE SYSTEM HIERARCHY & IMPORTANT DIRECTORIES

## 3.1 Theory

Unlike Windows (which uses drive letters like `C:\`, `D:\`), Linux has a single unified tree structure starting from the root directory, represented by a forward slash `/`. Every file, folder, and even connected device (USB drives, additional disks) is mounted somewhere inside this single tree. This standard is defined by the Filesystem Hierarchy Standard (FHS).

## 3.2 Why It Exists

A unified tree makes scripting and automation dramatically simpler. In Windows, a script must account for different drive letters. In Linux, a script can always assume /etc holds configuration and /var/log holds logs, regardless of what physical disk that data actually lives on. This predictability is exactly why tools like Ansible and Terraform can write generic automation that works across thousands of Linux servers.

## 3.3 The Complete Directory Tree Diagram
```bash
/
├── bin/        → Essential user command binaries (ls, cp, mv) [often symlinked to /usr/bin]
├── boot/       → Boot loader files, kernel images (vmlinuz), GRUB config
├── dev/        → Device files (disks, terminals) e.g. /dev/sda, /dev/null
├── etc/        → System-wide configuration files (nginx.conf, passwd, fstab)
├── home/       → Personal directories for each user (/home/ambika)
├── lib/        → Shared libraries needed by binaries in /bin and /sbin
├── media/      → Mount point for removable media (USB, CD-ROM)
├── mnt/        → Temporary mount point for sysadmins (manual mounts)
├── opt/        → Optional/third-party software (e.g. /opt/jenkins)
├── proc/       → Virtual filesystem exposing kernel & process info (not real files on disk)
├── root/       → Home directory for the root user (NOT the same as /)
├── run/        → Runtime data since last boot (PID files, sockets)
├── sbin/       → System binaries for admin tasks (reboot, iptables, fdisk)
├── srv/        → Data for services provided by this system (e.g. web server files)
├── tmp/        → Temporary files, cleared on reboot
├── usr/        → User-installed applications & their data (largest directory)
│   ├── bin/    → Non-essential user command binaries
│   ├── lib/    → Libraries for /usr/bin and /usr/sbin
│   └── local/  → Locally compiled/installed software
└── var/        → Variable data: logs, mail, spool, cache
    ├── log/    → System & application logs (/var/log/syslog, /var/log/nginx/)
    ├── www/    → Web server document root (common on Ubuntu: /var/www/html)
    └── lib/    → State information (databases, package manager data)
```
## 3.4 Deep Dive on the Directories You'll Use Daily

|Directory |What Lives Here |Why You Care (DevOps Angle)|
|---|---|---|
|`/etc` |All system & app config files |You will edit `/etc/nginx/nginx.conf`, `/etc/ssh/sshd_config`, `/etc/hosts constantly`|
|`/var/log` |All logs |First place to check when something breaks: `/var/log/syslog`, `/var/log/auth.log`|
|`/home` |User data |Your SSH keys often live in `/home/ubuntu/.ssh/`|
|`/opt` |Third-party apps |Jenkins, custom monitoring agents often install here|
|`/tmp` |Scratch space |Safe place to download/test scripts; wiped on reboot
|`/proc` |Live kernel/process data |`cat /proc/cpuinfo`, `cat /proc/meminfo` for hardware info without extra tools| 
|`/root` |Root's home |Only accessible via `sudo` — never store personal work here|
|`/var/www` |Web content |Where you'll deploy your Nginx HTML files (as you did in your EC2 project!)|

## 3.5 Step-by-Step: Navigating the File System

```bash
pwd                     # Print current working directory
ls                      # List files in current directory
ls -l                   # Long listing format (permissions, owner, size, date)
ls -la                  # Long listing + hidden files (dotfiles)
ls -lh                  # Long listing with human-readable sizes (KB, MB, GB)
cd /etc                 # Change directory to /etc
cd ..                   # Move up one directory
cd ~                    # Go to home directory
cd -                    # Go to previous directory
tree /etc -L 1          # Show directory tree, 1 level deep (install with: sudo apt install tree)
```
Sample Output of `ls -la`:
```bash
$ ls -la /home/ubuntu
total 32
drwxr-xr-x 5 ubuntu ubuntu 4096 Jul  1 10:22 .
drwxr-xr-x 3 root   root   4096 Jun 20 08:00 ..
-rw-r--r-- 1 ubuntu ubuntu  220 Jun 20 08:00 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu 3771 Jun 20 08:00 .bashrc
drwx------ 2 ubuntu ubuntu 4096 Jun 20 08:05 .ssh
-rw-r--r-- 1 ubuntu ubuntu  807 Jun 20 08:00 .profile
drwxrwxr-x 2 ubuntu ubuntu 4096 Jul  1 09:50 project
```
Reading this: the first column shows permissions (Chapter 4 explains this fully), then owner (`ubuntu`), group (`ubuntu`), size in bytes, last modified date, and filename. Notice `.ssh` has permissions `drwx------` — only the owner can access it, which is critical for SSH key security.

## 3.6 Absolute vs Relative Paths


- Absolute path: Starts from root `/`, always the same regardless of where you currently are. Example: `/var/www/html/index.html`
- Relative path: Starts from your current location. Example: if you're in `/var/www`, then `html/index.html` refers to the same file.


## 3.7 Best Practices


- Always use absolute paths inside scripts and cron jobs — relative paths break depending on where the script is executed from.
- Never store important personal data directly under `/` or `/root` — use `/home/<user>` or a dedicated `/data` mount.
- Regularly check `/var/log` disk usage — logs can fill up a disk unexpectedly (`du -sh /var/log/*`).


## 3.8 Common Mistakes


- Confusing `/root` (root user's home folder) with `/` (the actual root of the filesystem) — this is one of the most common beginner and interview mix-ups.
- Deleting files inside `/proc` thinking they're real files — they are virtual and represent live kernel state.
- Running scripts with relative paths from cron, which then fail because cron's working directory isn't what you expect.


## 3.9 Interview Questions


1. What is the Filesystem Hierarchy Standard (FHS)?
2. What is the difference between /root and /?
3. Where would you look for log files on a typical Ubuntu server?
4. What's the difference between /bin and /usr/bin, historically?
5. What is /proc, and why doesn't du correctly report its size like normal files?
6. Where should a third-party application like Jenkins typically be installed?


## 3.10 Scenario-Based Question

*Scenario:* You SSH into a new EC2 instance and are told "the application logs are filling up the disk." Walk through exactly which directories you'd check, which commands you'd run, and how you'd clean up safely without breaking the running application.

>(*Approach:* `df -h` to see which mount is full → `du -sh /var/log/* | sort -rh` to find biggest log directories → check if log rotation (`logrotate`) is configured → truncate or archive old logs rather than deleting files a process still has open, since that doesn't free disk space until the process is restarted.)

## 3.11 Troubleshooting


*Issue:* "No space left on device" error.

*Fix:* Run df -h to identify the full partition, then du -sh /var/* | sort -rh | head to find what's consuming space, typically /var/log or Docker images in /var/lib/docker.


## 3.12 Cheat Sheet

```bash
pwd            # Show current directory
ls -la         # List all files with details
cd <path>      # Change directory
tree -L 2      # Show tree structure, 2 levels
df -h          # Disk space by partition
du -sh <dir>   # Total size of a directory
```
## 3.13 Summary

The Linux file system is one unified tree rooted at /, with standardized directories for configs (/etc), logs (/var/log), user data (/home), and temporary files (/tmp). Knowing this map by heart lets you navigate any Linux server confidently, which is the single most tested practical skill in junior DevOps interviews.
