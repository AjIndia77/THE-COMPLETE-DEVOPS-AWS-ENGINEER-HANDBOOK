# CHAPTER 10: SYSTEM MONITORING — top, htop, free, df, du, netstat, ss

## 10.1 Theory

System monitoring commands let you observe CPU, memory, disk, and network usage in real time — essential for diagnosing performance issues, capacity planning, and confirming whether a server is healthy.

## 10.2 top — Real-Time Process Monitor
```sh
top
```
**Sample Output**:
```sh
top - 14:32:10 up 5 days,  2:15,  1 user,  load average: 0.15, 0.22, 0.18
Tasks: 110 total,   1 running, 109 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.2 us,  1.1 sy,  0.0 ni, 95.4 id,  0.2 wa,  0.0 hi,  0.1 si
MiB Mem :   1987.4 total,    412.1 free,    892.3 used,    683.0 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.    892.1 avail Mem

  PID USER   PR  NI    VIRT    RES    SHR S  %CPU  %MEM   TIME+ COMMAND
  892 www-data 20  0  55000   8900   3200 S   0.3   0.4   0:12.4 nginx
 1044 mysql    20  0 890000 210000  15000 S   0.7   9.8   4:22.1 mysqld
```
Key things to check: load average (should generally stay below your number of CPU cores), %CPU/%MEM usage per process (top offenders sort to top by default), and overall free memory.

Inside top (interactive keys):

```sh
P — sort by CPU usage
M — sort by memory usage
k — kill a process (enter PID)
q — quit
```

## 10.3 htop — A Friendlier, Colorized top

```bash
sudo apt install htop -y     # Install (Ubuntu)
sudo dnf install htop -y      # Install (RHEL/Amazon Linux)
htop
```
htop shows the same information as top but with color-coded bars for CPU/memory per core, mouse support, easier process searching (F3), and easier killing (F9). Most engineers prefer htop when available, but top is always pre-installed everywhere, so know both.

## 10.4 free — Memory Usage

```bash
free -h        # Human-readable (MB/GB instead of raw bytes)
free -m         # Show in megabytes specifically
```
**Sample Output**:
```sh
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           1.9Gi       892Mi       412Mi        45Mi       683Mi        1.0Gi
Swap:            0B          0B          0B
```
***Important concept***: "used" memory looks high, but Linux aggressively uses free RAM for disk caching (buff/cache) to speed things up — this cache is instantly reclaimed if an application needs it. The number that actually matters is the "available" column, not "used" or "free".

## 10.5 df — Disk Space (Filesystem Level)

```bash
df -h              # Human-readable disk space, all mounted filesystems
df -h /             # Check space on a specific mount point only
df -i                # Show inode usage (can run out of inodes even with free disk space!)
```
**Sample Output**:
```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G   12G  7.2G  63% /
tmpfs           989M     0  989M   0% /dev/shm
```
## 10.6 du — Disk Usage (Directory/File Level)

```bash
du -sh /var/log              # Total size of a specific directory (summary, human-readable)
du -sh /var/log/*              # Size of each item inside /var/log individually
du -sh /var/log/* | sort -rh     # Sort by size, largest first — great for finding what's eating disk space
du -h --max-depth=1 /var          # Show sizes one level deep only
```
**df vs du — Critical Distinction**:

| |df |du |
|---|---|---|
|Measures |Free/used space on the FILESYSTEM/partition |Space used by specific FILES/directories |
|Use case |"Is my disk full?" |"WHAT is filling my disk?" |
|Typical workflow |Run df -h first to spot the problem, THEN du -sh to find the cause |

## 10.7 netstat — Network Statistics (Legacy but Still Common)

```bash
netstat -tulnp        # Show all listening TCP/UDP ports with process names
# -t = TCP, -u = UDP, -l = listening only, -n = numeric (don't resolve hostnames), -p = show process/PID
```
**Sample Output**:
```sh
Proto Local Address     Foreign Address   State    PID/Program name
tcp   0.0.0.0:22         0.0.0.0:*         LISTEN   612/sshd
tcp   0.0.0.0:80         0.0.0.0:*         LISTEN   892/nginx
tcp   127.0.0.1:3306      0.0.0.0:*         LISTEN   1044/mysqld
```
This tells you: SSH (22) and Nginx (80) are listening on all interfaces, but MySQL (3306) is only listening on localhost (127.0.0.1) — meaning it's NOT accessible from outside the server, which is a good security practice.

## 10.8 ss — Modern Replacement for netstat

ss is faster than netstat and is the recommended modern tool (netstat is deprecated on many distros).

```bash
ss -tulnp             # Same flags, same output style as netstat
ss -t state established   # Show only established TCP connections
ss -s                       # Summary statistics
```
## 10.9 Real-World Example — Full Health Check Workflow

```bash
uptime                          # Quick load average + how long server's been up
free -h                          # Memory check
df -h                             # Disk check
top -bn1 | head -15                # One-time snapshot of top processes (good for scripts)
ss -tulnp                            # What ports/services are exposed
```
## 10.10 Architecture Diagram — Monitoring Decision Tree
```sh
        Server "feels slow" — where do you look first?
                        │
      ┌─────────────────┼─────────────────┐
      ▼                 ▼                 ▼
   CPU issue?       Memory issue?      Disk issue?
      │                 │                 │
      ▼                 ▼                 ▼
  top / htop          free -h           df -h
  (check load avg,   (check available   (check % used)
   sort by %CPU)      not "used")           │
                                             ▼
                                          du -sh /var/*
                                       (find the culprit dir)
```
## 10.11 Best Practices


- Always check the "available" column in free -h, not "free" — "free" is misleadingly low due to healthy disk caching.
- Run df -h BEFORE du -sh when disk is full — df tells you WHERE the problem is, du tells you WHAT is causing it.
- Set up automated alerting (CloudWatch in AWS, covered in Part 2) rather than manually checking top — manual checks don't scale.
- Check inode usage (df -i) if you get "No space left on device" even when df -h shows free space — you can run out of inodes with millions of tiny files.


## 10.12 Common Mistakes


- Panicking over high "used" memory in free -h without checking "available" first.
- Using netstat on newer minimal distros where it's not installed by default — reach for ss instead.
- Forgetting -h and staring at disk sizes in raw bytes, unable to quickly interpret them.
- Not knowing that du and df can report DIFFERENT numbers when a process is still holding a deleted file open (the disk space isn't freed until the process restarts/closes the file handle).


## 10.13 Interview Questions


1. What's the difference between df and du?
2. Why might free -h show high "used" memory even on a healthy server?
3. How would you find the top 5 largest subdirectories inside /var/log?
4. What's the difference between netstat and ss, and why is ss generally preferred now?
5. How would you check which process is listening on port 8080?
6. What does "load average" in top/uptime actually represent?


## 10.14 Scenario-Based Question

**Scenario**: df -h shows / is 98% full, but running du -sh /* and adding up the numbers only accounts for 60% of the disk. Where is the missing space, and how do you find it?

> (Approach: This is the classic "deleted-but-open-file-handle" problem — a process (often a log file) was deleted while still open by a running process, so the space isn't released. Find it with lsof +L1 or lsof | grep deleted, identify the PID, and restart that service to release the space.)

## 10.15 Troubleshooting


**Issue**: Server unresponsive/slow, unclear why.

**Fix**: Run uptime (load average), free -h (memory), df -h (disk), top (CPU hogs) — in that order, takes under a minute and covers 90% of causes.

**Issue**: Port appears closed even though the service is "running."

**Fix**: ss -tulnp | grep <port> to confirm what's actually listening and on which interface (0.0.0.0 vs 127.0.0.1 matters!).


## 10.16 Cheat Sheet

```bash
top                    # Real-time process monitor
htop                     # Friendlier version of top
free -h                   # Memory usage (human-readable)
df -h                      # Disk space by filesystem
du -sh dir/                 # Total size of a directory
du -sh dir/* | sort -rh      # Find biggest subdirectories
netstat -tulnp                # Listening ports (legacy)
ss -tulnp                      # Listening ports (modern)
uptime                          # Load average + uptime
```
## 10.17 Summary

top/htop, free, df/du, and netstat/ss together form your complete "server health dashboard" from the command line. In any performance-related interview question or real incident, your answer should follow the same pattern: check load → check memory → check disk → check network — this chapter gives you every command for each step.
