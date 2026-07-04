# CHAPTER 4: FILE PERMISSIONS, chmod, chown

## 4.1 Theory

Linux is a multi-user operating system, meaning many people (or processes) can use the same machine simultaneously. To prevent one user from reading, modifying, or deleting another user's files, Linux enforces a permission system on every single file and directory.

Every file has three permission categories applied to three types of users:

### Permission Types:


- r (read) = 4 — view file contents / list directory contents
- w (write) = 2 — modify file / add-remove files in directory
- x (execute) = 1 — run file as a program / enter (`cd` into) a directory


### User Categories:


- Owner (u) — the user who owns the file
- Group (g) — users belonging to the file's assigned group
- Others (o) — everyone else on the system


## 4.2 Why It Exists

Imagine a shared web server where 5 developers have accounts. Without permissions, any developer could accidentally (or maliciously) delete another's code, read another user's SSH private keys, or overwrite production configuration files. Permissions enforce least privilege — a core security principle you'll be tested on in every AWS interview too (IAM policies work on the exact same philosophy).

## 4.3 Reading Permission Strings
```bash
-rwxr-xr--  1 user devops  1024  Jul 1 10:00  deploy.sh
```
Breaking this down character by character:
```bash
 -    rwx    r-x    r--
 │     │      │      │
 │   Owner  Group  Others
 │
 File Type (- = regular file, d = directory, l = symlink)
```

- Type: `-` → regular file
- Owner (`user`): `rwx` → read, write, execute (4+2+1 = 7)
- Group (`devops`): `r-x` → read, execute, no write (4+0+1 = 5)
- Others: `r--` → read only (4+0+0 = 4)


So numerically this permission is **754**.

## 4.4 Architecture Diagram — Permission Decision Flow
```bash
              User tries to access a file
                        │
                        ▼
          Is the user the OWNER of the file?
             │YES                  │NO
             ▼                     ▼
     Apply OWNER perms    Is user in the file's GROUP?
                              │YES             │NO
                              ▼                ▼
                      Apply GROUP perms   Apply OTHERS perms
```
## 4.5 chmod — Changing Permissions

Two ways to use chmod: Numeric (octal) and Symbolic

Numeric method:
```bash
chmod 755 deploy.sh     # rwxr-xr-x — owner full, group/others read+execute
chmod 644 config.txt    # rw-r--r-- — owner read/write, everyone else read-only
chmod 700 private.sh    # rwx------ — only owner can do anything
chmod 600 id_rsa        # rw------- — standard permission for SSH private keys
chmod -R 755 /var/www/html   # -R = recursive, apply to all files/subfolders
```
Symbolic method:

```bash
chmod u+x script.sh      # Add execute permission for owner (u=user/owner)
chmod g-w file.txt        # Remove write permission from group
chmod o=r file.txt        # Set others' permission to read-only exactly
chmod a+r file.txt        # Add read for all (a = all: u+g+o)
chmod u+x,g+x script.sh   # Add execute for both owner and group
```
Sample Output — Before and After:
```bash
$ ls -l deploy.sh
-rw-r--r-- 1 ambika ambika 512 Jul 1 10:00 deploy.sh

$ chmod +x deploy.sh
$ ls -l deploy.sh
-rwxr-xr-x 1 ambika ambika 512 Jul 1 10:00 deploy.sh
```
## 4.6 chown — Changing Ownership

`chown` changes who owns a file (both user and/or group).

```bash
chown ambika file.txt              # Change owner to 'ambika'
chown ambika:devops file.txt        # Change owner to 'ambika' AND group to 'devops'
chown :devops file.txt              # Change only the group to 'devops'
chown -R www-data:www-data /var/www/html  # Recursively change ownership (common for web servers)
```
Real-World DevOps Example: When you deploy a website with Nginx (as you did in your EC2 project), Nginx runs as user `www-data` on Ubuntu. If your HTML files are owned by ubuntu with restrictive permissions, Nginx may return a 403 Forbidden error because the `www-data` user cannot read your files. The fix:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```
## 4.7 chgrp — Changing Group Only

```bash
chgrp devops file.txt   # Equivalent to: chown :devops file.txt
```
## 4.8 Special Permissions (Advanced)

|Special Permission |Numeric |Symbol |Purpose |
|---|---|---|---|
|SUID (Set User ID) |4000 |s in owner's execute slot |Program runs with the file owner's privileges, not the person running it (e.g., /usr/bin/passwd runs as root even when a normal user runs it, so it can update /etc/shadow) |
|SGID (Set Group ID) |2000 |s in group's execute slot |Files created inside a directory inherit the directory's group |
|Sticky Bit |1000 |t in others' execute slot |In a shared directory (like /tmp), users can only delete their OWN files, even if others have write access|

```bash
chmod 4755 /usr/bin/somebinary   # Set SUID
chmod 2775 /shared/team_folder    # Set SGID on a shared team directory
chmod 1777 /tmp                   # Sticky bit (this is actually /tmp's default!)
```
## 4.9 Best Practices


- Never use `chmod 777` on production files — it grants read/write/execute to literally everyone, a massive security hole. This is one of the most common mistakes junior engineers make and one of the most common interview "gotcha" questions.
- Always use `chmod 600` for private SSH keys (`.pem`, `id_rsa`) — SSH will actually refuse to use keys with looser permissions.
- Use groups for team collaboration instead of loosening `"others"` permissions.
- Use `-R` carefully — recursive chmod/chown on the wrong directory (like / by mistake) can break your entire system.


## 4.10 Common Mistakes


- Running chmod `777` "just to make it work" instead of diagnosing the actual permission problem.
- Forgetting that directories need execute (`x`) permission to be entered with cd, not just read — a very common source of confusion.
- Changing ownership of system files (like `/etc/passwd`) by mistake, which can lock you out of the system.
- Confusing `chmod` (permissions) with `chown` (ownership) — different commands, different jobs.


## 4.11 Interview Questions


1. Explain the difference between `chmod` and `chown`.
2. What does permission `755` mean in symbolic form?
3. Why does SSH refuse to connect if your private key has permission `644`?
4. What is the `SUID` bit, and can you give a real example of where Linux uses it?
5. Why is chmod `777` considered dangerous?
6. What permission does a directory need for a user to `cd` into it?
7. How would you recursively give a web server user ownership of a directory?


## 4.12 Scenario-Based Question

**Scenario:** You deploy a website to /var/www/html on a fresh Ubuntu EC2 instance. When you visit the site in a browser, you get a "403 Forbidden" error even though the files exist and Nginx is running. Walk through your troubleshooting steps.

>(**Approach:** Check file ownership ls -la /var/www/html → confirm owner/group is www-data or that "others" has read+execute → check directory permissions specifically need execute bit for Nginx to traverse into them → check for a missing index.html → check Nginx error log at /var/log/nginx/error.log for the exact permission denial.)

## 4.13 Troubleshooting


- **Issue:** `Permission denied (publickey)` when SSHing.
**Fix:** chmod 400 or chmod 600 your .pem/id_rsa file locally before use.
- **Issue:** 403 Forbidden from Nginx/Apache.
**Fix:** Check both file AND parent directory permissions; web server user needs execute on every directory in the path down to the file.


## 4.14 Cheat Sheet

```bash
chmod 755 file            # rwxr-xr-x
chmod 644 file            # rw-r--r--
chmod 600 file            # rw-------  (SSH keys)
chmod +x file              # Add execute for all
chmod -R 755 dir/          # Recursive
chown user file            # Change owner
chown user:group file       # Change owner and group
chown -R user:group dir/    # Recursive ownership change
chgrp group file            # Change group only
ls -l                       # View permissions
stat file                   # Detailed permission + timestamp info
```
## 4.15 Summary

File permissions are Linux's core security mechanism, controlled through the read/write/execute triad for owner/group/others. chmod changes what actions are allowed; chown changes who is allowed to take them. Mastering this — especially avoiding the chmod 777 trap — is non-negotiable for any DevOps role and is guaranteed interview material.
