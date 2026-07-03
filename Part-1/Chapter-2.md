# CHAPTER 2: LINUX ARCHITECTURE

## 2.1 Theory

Linux architecture is layered, moving from hardware at the bottom to user applications at the top. The four major layers are:


- Hardware — physical CPU, RAM, disk, network interface.
- Kernel — the core software layer that manages hardware resources. It has four major responsibilities:

    1. Process Management — deciding which program gets CPU time and when (scheduling).
    2. Memory Management — allocating and freeing RAM to processes.
    3. Device Management — talking to hardware via drivers.
    4. File System Management — organizing how data is stored and retrieved on disk.



- Shell — a command-line interpreter (bash, sh, zsh) that takes the commands you type and translates them into kernel system calls.
- Applications — the actual programs you run (Docker, Nginx, Python scripts, Jenkins).


## 2.2 Why It Exists

This layered design exists for separation of concerns and security. Regular applications never talk to hardware directly — they must go through the kernel. This prevents a buggy or malicious program from crashing your entire machine or reading another program's memory directly. This is also why Linux is considered more stable and secure than direct-hardware-access architectures.

## 2.3 Real-World Example

When you run `ls` in your terminal:


The shell (bash) reads your input `ls`.

Bash finds the `ls` binary (usually at `/bin/ls`) using the PATH environment variable (covered in Chapter 13).

The shell asks the kernel to create a new process for `ls`.

The kernel allocates memory, schedules CPU time, and lets `ls` read directory data from the file system.

Output is sent back through the kernel to your terminal screen.


## 2.4 Architecture Diagram — Kernel Space vs User Space
```bash

                    USER SPACE
   ┌───────────┐  ┌───────────┐  ┌───────────┐
   │   bash    │  │  Docker   │  │  Nginx    │
   └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
         │  System Calls (open, read, write, fork, exec)
─────────┼──────────────┼──────────────┼──────────────
         ▼              ▼              ▼
                    KERNEL SPACE
   ┌─────────────────────────────────────────┐
   │  Process Scheduler | Memory Manager       │
   │  File System (ext4, xfs) | Network Stack  │
   │  Device Drivers                           │
   └─────────────────────────────────────────┘
                        │
                        ▼
                    HARDWARE
             (CPU, RAM, Disk, NIC)
```

## 2.5 Monolithic Kernel Concept

Linux uses a monolithic kernel design (as opposed to a microkernel). This means most core services (file systems, device drivers, network stack) run in kernel space for performance, but Linux also supports Loadable Kernel Modules (LKMs) — pieces of code that can be added or removed from the kernel at runtime without rebooting (e.g., a new network card driver).
```bash
# View loaded kernel modules
lsmod
```

### View kernel version
`uname -r`

### View full system information
`uname -a`

Sample Output:

`$ uname -a`

> Linux ip-172-31-20-15 5.15.0-1041-aws #46-Ubuntu SMP x86_64 GNU/Linux

> This tells you: kernel name (Linux), hostname (ip-172-31-20-15 — a typical AWS EC2 auto-generated hostname), kernel version (5.15.0-1041-aws — notice the "-aws" suffix showing this is an AWS-optimized kernel), and architecture (x86_64).

## 2.6 Best Practices


- Always check uname -a and cat /etc/os-release when you SSH into an unfamiliar server for the first time — never assume the OS.
- Avoid loading/unloading kernel modules on production servers unless you fully understand the impact.


## 2.7 Common Mistakes


Confusing "kernel space" and "user space" — a very common conceptual interview trap.

Believing you need to understand kernel internals deeply to be a DevOps engineer — you need working knowledge, not kernel-developer-level depth.


## 2.8 Interview Questions


1. What are the four main functions of the Linux kernel?
2. What is the difference between kernel space and user space?
3. What is a system call? Give an example.
4. What is a monolithic kernel, and how does Linux support modularity within one?
5. How would you check your current kernel version on a live server?


## 2.9 Scenario-Based Question

Scenario: A production EC2 instance is crashing intermittently. Your senior engineer suspects a bad kernel module was loaded after a recent update. How would you check loaded modules and roll back to a previous kernel version if needed?

## 2.10 Troubleshooting


*Issue:* Server won't boot after a kernel update.

*Fix:* Use the GRUB bootloader menu to select the previous working kernel version (on AWS, this may require attaching the EBS volume to another instance to fix boot configs).


## 2.11 Cheat Sheet
```bash
bashuname -a          # Full kernel & system info
uname -r           # Kernel version only
cat /etc/os-release # Distro name and version
lsmod               # List loaded kernel modules
```

## 2.12 Summary

Linux architecture is a layered system where the kernel mediates all access between applications and hardware. Understanding this layering — especially kernel space vs. user space — gives you the mental model needed to troubleshoot everything from a crashing process to a slow disk.
