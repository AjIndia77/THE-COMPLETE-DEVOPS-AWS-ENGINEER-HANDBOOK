# CHAPTER 5: USERS, GROUPS, sudo, passwd, USER & GROUP MANAGEMENT

## 5.1 Theory

Every process and file on Linux belongs to a user. Users are further organized into groups to simplify permission management for teams. There are three categories of users:


1. root — the superuser, UID (User ID) 0, has unrestricted access to everything on the system.
2. System users — created automatically by services (e.g., www-data, mysql, jenkins) to run those services with limited privileges, not meant for human login.
3. Regular users — human accounts, typically UID 1000 and above.


## 5.2 Why It Exists

Running everything as root all the time is a massive security risk — one mistake or one compromised process could destroy the entire system. The user/group model, combined with sudo, lets regular users perform administrative tasks only when needed and only with explicit elevation, following the principle of least privilege.

## 5.3 Key Files Behind User Management

|File |Purpose|
|---|---|
|`/etc/passwd` |List of all users (username, UID, GID, home dir, shell) — readable by everyone|
|`/etc/shadow` |Encrypted passwords — readable ONLY by root/etc/groupList of all groups and their members|
|`/etc/sudoers` |Defines who can use sudo and what they're allowed to run|

```bash
cat /etc/passwd | head -3
```
### Sample Output:
```sh
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
```
Reading this line by line (colon-separated fields): `username:password-placeholder:UID:GID:comment:home-directory:default-shell`. The x means the real password is stored securely in `/etc/shadow`. Notice daemon has shell `/usr/sbin/nologin` — this means it's a system account that can never be used to log in interactively, which is a security best practice for service accounts.

## 5.4 User Management Commands

```bash
# Create a new user with a home directory
sudo useradd -m -s /bin/bash newdev

# Create a user AND set additional options (comment, groups)
sudo useradd -m -c "New Developer" -s /bin/bash -G sudo,devops newdev

# Set/change a password for a user
sudo passwd newdev

# Modify an existing user (add to a group, change shell, etc.)
sudo usermod -aG docker newdev     # -aG = append to supplementary group (Docker access!)
sudo usermod -s /bin/zsh newdev     # Change default shell

# Delete a user
sudo userdel newdev                 # Deletes user but keeps home directory
sudo userdel -r newdev              # Deletes user AND home directory

# Switch to another user
su - newdev                         # Full login shell as newdev (asks for newdev's password)
sudo su -                           # Switch to root (uses YOUR password via sudo)

# View currently logged-in user
whoami
id                                  # Shows UID, GID, and all groups you belong to
```
### Sample Output of id:
```bash
$ id user
uid=1001(user) gid=1001(user) groups=1001(user),27(sudo),999(docker)
```
This tells you `user` is in the sudo group (can run admin commands) and the docker group (can run Docker without sudo — very common real-world DevOps setup).

5.5 Group Management Commands

```bash
sudo groupadd devops                 # Create a new group
sudo groupdel devops                 # Delete a group
sudo usermod -aG devops user       # Add existing user to a group
sudo gpasswd -d user devops        # Remove a user from a group
getent group devops                  # Show members of a specific group
cat /etc/group | grep devops         # Alternative way to check group membership
groups user                        # List all groups a user belongs to
```
## 5.6 sudo — Superuser Do

sudo allows a permitted user to run a single command with root (or another user's) privileges, temporarily, and it logs every action for auditing — unlike su which switches your entire session to another user.

```bash
sudo apt update                      # Run one command as root
sudo -i                              # Start an interactive root shell
sudo -u www-data whoami              # Run a command as a SPECIFIC user (not just root)
sudo visudo                          # SAFELY edit /etc/sudoers (checks syntax before saving!)
```
**Configuring sudo access** — inside /etc/sudoers (edited only via visudo, never directly):
```bash
# Give full sudo access to a user
user  ALL=(ALL:ALL) ALL

# Give passwordless sudo (common for automation/CI users)
jenkins ALL=(ALL) NOPASSWD: ALL

# Give a user sudo access to only specific commands
deploy  ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```
## 5.7 passwd — Password Management

```bash
passwd                        # Change YOUR OWN password
sudo passwd ambika            # Change another user's password (admin only)
sudo passwd -l ambika         # Lock a user account (disable login)
sudo passwd -u ambika         # Unlock a user account
sudo chage -l ambika           # Show password aging/expiry info
sudo chage -M 90 ambika        # Force password to expire every 90 days
```
## 5.8 Architecture Diagram — Authentication Flow
```bash
   User types: sudo apt update
              │
              ▼
    sudo checks /etc/sudoers ─── Is user authorized? ── NO ──► "user is not in the sudoers file"
              │ YES
              ▼
    Prompts for USER's OWN password (not root's password)
              │
              ▼
    Password validated against /etc/shadow
              │
              ▼
    Command runs with root (UID 0) privileges
              │
              ▼
    Action logged to /var/log/auth.log (Ubuntu) or /var/log/secure (RHEL)
```
## 5.9 Best Practices


- Never log in directly as root over SSH — disable root SSH login (PermitRootLogin no in /etc/ssh/sshd_config) and use sudo instead, so every privileged action is logged and traceable to a real user.
- Always use visudo to edit sudoers — it validates syntax and prevents you from locking yourself out with a typo.
- Add automation users (like a Jenkins service account) to sudo with NOPASSWD scoped to ONLY the specific commands they need, never ALL.
- Use groups (docker, sudo, devops) to manage permissions for teams rather than editing individual user permissions repeatedly.

## 5.10 Common Mistakes

- Editing /etc/sudoers directly with vim or nano instead of visudo — a syntax error here can lock out ALL sudo access on the system.
- Using su instead of sudo in shared/production environments, losing the audit trail of who did what.
- Forgetting that after usermod -aG docker <user>, the user must log out and back in for the new group membership to take effect in their current session.
- Giving NOPASSWD: ALL sudo access carelessly to automation accounts, creating a major security risk if that account is ever compromised.


## 5.11 Interview Questions


1. What is the difference between su and sudo?
2. Where are user passwords actually stored, and why isn't it in /etc/passwd?
3. How do you add an existing user to the docker group, and why would you need to?
4. What does visudo do differently from editing /etc/sudoers with a normal text editor?
5. How do you lock a user account without deleting it?
6. What is the difference between a system account and a regular user account?
7. Why is disabling root SSH login considered a security best practice?


## 5.12 Scenario-Based Question

Scenario: A new DevOps intern joins your team. They need to: (1) SSH into the shared staging server, (2) run Docker commands without typing sudo every time, (3) NOT have full root access. Walk through every command you'd run to set this up correctly.

> (Approach: sudo useradd -m -s /bin/bash internname → sudo passwd internname (or set up SSH key auth instead) → sudo usermod -aG docker internname → intern logs out/in → verify with groups internname and a test docker ps command without sudo.)

## 5.13 Troubleshooting

***Issue***: "username is not in the sudoers file. This incident will be reported."

***Fix***: Log in as an existing sudo user (or root) and add the user via usermod -aG sudo <username> (Ubuntu) or usermod -aG wheel <username> (RHEL/CentOS).

***Issue***: Docker commands still need sudo after adding user to the docker group.

***Fix***: The user must log out and log back in (or run newgrp docker) for group changes to apply to the active session.


## 5.14 Cheat Sheet

```bash
useradd -m -s /bin/bash user     # Create user with home dir
passwd user                       # Set password
usermod -aG group user            # Add user to a group
userdel -r user                   # Delete user + home dir
groupadd group                    # Create group
id user                           # Show UID/GID/groups
whoami                            # Current user
su - user                         # Switch user (full login)
sudo -i                           # Root shell
visudo                            # Safely edit sudoers
chage -l user                     # Password expiry info
```
## 5.15 Summary

Linux's multi-user model, enforced through /etc/passwd, /etc/shadow, and /etc/group, combined with sudo for controlled privilege escalation, is the backbone of server security. As a DevOps engineer, you'll constantly create service accounts, manage group-based access (especially for Docker), and configure sudo rules for automation — get comfortable with every command in this chapter.
