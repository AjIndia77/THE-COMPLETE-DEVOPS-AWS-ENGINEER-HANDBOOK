# CHAPTER 1: INTRODUCTION TO LINUX

## 1.1 Theory

Linux is a free, open-source operating system kernel originally created by Linus Torvalds in 1991 as a personal project while he was a student at the University of Helsinki, Finland. Unlike Windows or macOS, which are owned by single companies (Microsoft and Apple), Linux is developed collaboratively by thousands of programmers around the world and is released under the GNU General Public License (GPL), meaning anyone can view, modify, and redistribute its source code.

Strictly speaking, "Linux" refers only to the kernel — the core program that talks directly to your computer's hardware (CPU, memory, disk, network card). What most people call "Linux" (like Ubuntu, CentOS, Red Hat Enterprise Linux, Fedora, Debian) is actually GNU/Linux — the Linux kernel bundled together with thousands of GNU tools, utilities, and a package manager, all wrapped into something called a distribution (or "distro").

## 1.2 Why Linux Exists (and Why It Matters for DevOps)

Windows dominates personal desktops, but Linux dominates servers. Over 90% of the world's cloud infrastructure (including AWS EC2 instances), nearly all of the top 500 supercomputers, and the majority of Docker containers run on Linux. Here's why:


Free and open-source — no licensing cost, which matters enormously at cloud scale (imagine paying a Windows license fee for 10,000 servers).
Stability — Linux servers can run for years without a reboot.
Security — a robust permission model and a smaller attack surface compared to GUI-heavy operating systems.
Customizability — you can strip Linux down to only what you need (critical for lightweight Docker images).
Automation-friendly — Linux is built around the command line, which is exactly what tools like Ansible, Terraform, and Jenkins need to operate.


For you as a future DevOps engineer: Every AWS EC2 instance you'll launch in Part 2 of this book will most likely run a Linux distribution (Amazon Linux 2/2023, Ubuntu, or RHEL). Every Docker container is built on a minimal Linux base image. If you don't know Linux, you cannot do DevOps — period.

## 1.3 Real-World Example

Imagine you are hired as a Cloud/DevOps Intern. On Day 1, your manager says: "SSH into the staging EC2 instance, check why the Nginx service is down, look at the logs, and restart it if needed." That single sentence requires you to know: SSH, systemctl, journalctl, and basic file navigation — all covered in this Part.

## 1.4 Popular Linux Distributions (Distros)

|🦾 Distro |Based On |Package Manager |Common Use Case|
|---|---|---|---|
|1. Ubuntu |Debian |apt(.deb) |Cloud servers, general purpose, most beginner-friendly
|2. Debian |—(original) |apt(.deb) |Stability-focused servers
|3. Red Hat Enterprise Linux (RHEL) |—(original) |dnf/yum (.rpm) |Enterprise, banking, government (paid support)
|4. CentOS / CentOS Stream |RHEL |dnf/yum(.rpm) |Free RHEL alternative (widely used before CentOS 8 EOL)
|5. Amazon Linux2/2023 |RHEL/Fedora |dnf/yum(.rpm) |AWS EC2 default, optimized for AWS
|6. Fedora |—(original) |dnf(.rpm) |Cutting-edge desktop/testing ground for RHEL
|7. Kali Linux |Debian |apt(.deb) |Penetration testing, security research


Important for your RHCSA background: Since you're RHCSA V9 and V10 certified, you already have strong RPM-based (Red Hat family) experience. This book will cover both the Debian family (apt) and Red Hat family (dnf/yum) commands side by side, since AWS interviews often expect you to know both — Amazon Linux is RPM-based, but Ubuntu (Debian-based) is the most common EC2 choice in the industry.

## 1.5 Architecture Diagram — Where Linux Fits
```sh

 ┌─────────────────────────────────────────────┐
 │              USER APPLICATIONS               │
 │   (Jenkins, Docker, Nginx, MySQL, Python)     │
 ├─────────────────────────────────────────────┤
 │              SHELL (bash, zsh)                │
 │   (interprets commands you type)              │
 ├─────────────────────────────────────────────┤
 │         GNU UTILITIES / SYSTEM LIBRARIES       │
 │   (ls, cp, mv, grep, glibc, systemd)          │
 ├─────────────────────────────────────────────┤
 │              LINUX KERNEL                     │
 │  (Process Mgmt, Memory Mgmt, Device Drivers,  │
 │   File System, Network Stack)                 │
 ├─────────────────────────────────────────────┤
 │              HARDWARE                         │
 │   (CPU, RAM, Disk, Network Card, EC2 vCPU)    │
 └─────────────────────────────────────────────┘
 ```

## 1.6 Best Practices


Always use a well-supported LTS (Long Term Support) distro version in production (e.g., Ubuntu 22.04 LTS, not a random interim release).
Keep your system updated regularly (apt update && apt upgrade / dnf update).
Learn one distro family deeply first (you already know RHEL family), then cross-train on the other (Debian/Ubuntu) since most job postings mention Ubuntu.


## 1.7 Common Mistakes


Assuming all Linux distros use the same package manager — they don't (apt vs dnf/yum).
Confusing "Linux" (the kernel) with "Linux distribution" (the full OS) in interviews — this is a common trick question.
Thinking Linux has no GUI — most distros do, but servers typically run headless (no GUI) to save resources.


## 1.8 Interview Questions


What is the difference between Linux (the kernel) and a Linux distribution?
Why do most cloud servers run Linux instead of Windows?
Name three RPM-based and three Debian-based distributions.
What license is Linux released under, and what does that mean practically?
What is Amazon Linux, and why would AWS create its own distro?


## 1.9 Scenario-Based Question

Scenario: Your company's DevOps team is standardizing all EC2 instances on one OS family. The current stack has a mix of Ubuntu and CentOS Stream servers, causing confusion because engineers keep running apt on CentOS and yum on Ubuntu by mistake. How would you propose standardizing this, and what documentation would you create?

(Sample answer approach: Recommend Ubuntu LTS for its larger community/AWS AMI support, create an internal wiki page mapping every yum/dnf command engineers know to its apt equivalent, and use Infrastructure as Code (Terraform, covered in Part 4) to enforce a single AMI going forward.)

## 1.10 Troubleshooting


Issue: "Command not found" errors when switching from Ubuntu to CentOS or vice versa.
Fix: Check which package manager the distro uses (cat /etc/os-release tells you the distro), then use the correct manager (see Chapter 14).


## 1.11 Summary

Linux is the operating system that powers the cloud. Understanding what Linux is, how distributions differ, and why the industry standardized on it will make every future chapter — permissions, networking, Docker, AWS — make sense in context.