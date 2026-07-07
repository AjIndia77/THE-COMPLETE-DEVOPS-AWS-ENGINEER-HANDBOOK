## 🧐 TABLE OF CONTENTS — PART 1


1. [Introduction to Linux](./Chapter-01.md)
2. [Linux Architecture](./Chapter-02.md)
3. [File System Hierarchy & Important Directories](./Chapter-03.md)
4. [File Permissions, chmod, chown](./Chapter-04.md)
5. [Users, Groups, sudo, passwd](./Chapter-05.md)
6. [Process Management, Services, systemctl, journalctl, cron](./Chapter-06.md)
7. [Networking Commands, SSH, SCP, rsync](./Chapter-07.md)
8. [Archiving: tar, zip](./Chapter-08.md)
9. [Text Processing: grep, awk, sed, find, locate](./Chapter-09.md)
10. [System Monitoring: top, htop, free, df, du, netstat, ss](./Chapter-10.md)
11. [curl, wget](./Chapter-11.md)
12. [Text Editors: vim, nano](./Chapter-12.md)
13. [Bash Scripting & Environment Variables](./Chapter-13.md)
14. [Package Management: apt, snap, Repositories](./Chapter-14.md)



## 😎 PART 1 MASTER CHEAT SHEET

```bash
### NAVIGATION ###
pwd | ls -la | cd <dir> | cd .. | cd ~

### PERMISSIONS ###
chmod 755 file | chmod +x file | chown user:group file | chmod -R 755 dir/

### USERS & GROUPS ###
useradd -m -s /bin/bash user | passwd user | usermod -aG group user
userdel -r user | groupadd group | id user | sudo visudo

### PROCESSES & SERVICES ###
ps aux | kill -9 <PID> | systemctl start|stop|restart|status <svc>
systemctl enable --now <svc> | journalctl -u <svc> -f | crontab -e

### NETWORKING / SSH ###
ssh -i key.pem user@ip | scp -i key.pem file user@ip:/path
rsync -avz src/ user@ip:/dest/ | ping -c 4 host | ip a | ss -tulnp

### ARCHIVES ###
tar -czvf out.tar.gz dir/ | tar -xzvf out.tar.gz | zip -r out.zip dir/ | unzip out.zip

### TEXT PROCESSING ###
grep -rn "text" dir/ | awk '{print $1}' file | sed -i 's/old/new/g' file
find . -name "*.log" -mtime -7 | locate filename

### MONITORING ###
top | htop | free -h | df -h | du -sh dir/* | sort -rh

### WEB REQUESTS ###
curl -I url | curl -O url | wget url | wget -c url

### EDITORS ###
vim file (i / Esc / :wq / :q!) | nano file (Ctrl+O save, Ctrl+X exit)

### SCRIPTING ###
#!/bin/bash | chmod +x script.sh | ./script.sh
if [ cond ]; then ... fi | for i in list; do ... done | export VAR=value

### PACKAGE MANAGEMENT ###
apt update && apt upgrade -y | apt install <pkg> -y | dnf install <pkg> -y | snap install <pkg>
```


## 🦾 PART 1 — FULL INTERVIEW QUESTION BANK (Quick Revision)

**Conceptual:**

1. Difference between Linux kernel and a distro?
2. Explain kernel space vs user space.
3. What is the FHS, and name 5 key directories.
4. Explain the permission triad (owner/group/others) with an example.
5. Difference between su and sudo.
6. What is systemd, and what is PID 1?
7. Difference between SIGTERM and SIGKILL.
8. Explain how SSH key-based authentication works.
9. Difference between df and du.
10. Difference between apt update and apt upgrade.

**Command-Based (Practical):**

11. Write a command to recursively change ownership of /var/www/html to www-data.
12. Write a cron expression to run a script every day at 2:30 AM.
13. Write a command to find all files larger than 100MB modified in the last 7 days.
14. Write a sed command to replace "8080" with "9090" in a config file, saving changes.
15. Write a bash script that checks if Nginx is running and restarts it if not.
16. How do you add a user to the docker group?
17. How do you check which process is using a specific port?
18. Write an awk command to find the top memory-consuming process.
19. How do you safely edit /etc/sudoers?
20. How do you install Docker's official repository on Ubuntu?

**Scenario-Based (Most Common in Real Interviews):**

21. Production server disk is full — walk through your exact troubleshooting steps.
22. A deployed service doesn't survive a reboot — what did you likely forget, and how do you fix it?
23. You get "Permission denied (publickey)" connecting via SSH — list every possible cause and fix.
24. You need to deploy code to 5 servers efficiently — what approach and tools would you use?
25. A website returns 403 Forbidden after deployment — diagnose step by step.



## 🙌 PART 1 — FINAL SUMMARY

You've now covered the complete foundation every DevOps and Cloud engineer needs before touching AWS: how Linux is architected, how the file system is organized, how permissions and users protect the system, how to manage processes and services with systemctl/journalctl/cron, how to connect securely with SSH and move files with scp/rsync, how to archive and search through data with tar/grep/awk/sed/find, how to monitor system health, how to fetch data over the web with curl/wget, how to edit files with vim/nano, how to automate with bash scripting, and how to manage software with apt/dnf/snap.

Every single AWS EC2 instance, Docker container, and Jenkins server you build in the rest of this book will depend on the skills in this Part. Before moving to Part 2 — AWS Cloud Fundamentals, make sure you can comfortably: SSH into a server, navigate the file system, check and fix permissions, manage a systemd service, write a basic bash deployment script, and install software with the correct package manager — all without looking anything up.

Next up: [Part 2]()— AWS Cloud Fundamentals (EC2, VPC, S3, IAM, Security Groups, and the AWS CLI).

