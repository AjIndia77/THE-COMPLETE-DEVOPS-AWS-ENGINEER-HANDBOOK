# CHAPTER 8: ARCHIVING — tar, zip

## 8.1 Theory

Archiving combines multiple files/directories into a single file, often with compression to reduce size. Linux primarily uses tar (Tape Archive) combined with compression algorithms like gzip or bzip2; zip is also available for cross-platform compatibility (especially with Windows users).

## 8.2 Why It Exists

Transferring thousands of small files individually over a network is slow (each file has connection overhead). Archiving bundles everything into one file for faster transfer, easier backup, and simpler storage — critical when backing up application code, logs, or preparing deployment packages.

## 8.3 tar Commands

```bash
tar -cvf archive.tar myfolder/       # Create an archive (no compression)
tar -xvf archive.tar                  # Extract an archive
tar -czvf archive.tar.gz myfolder/     # Create a GZIP-compressed archive
tar -xzvf archive.tar.gz                # Extract a GZIP-compressed archive
tar -cjvf archive.tar.bz2 myfolder/      # Create a BZIP2-compressed archive (smaller, slower)
tar -xjvf archive.tar.bz2                 # Extract a BZIP2 archive
tar -tvf archive.tar.gz                    # List contents WITHOUT extracting
tar -xzvf archive.tar.gz -C /destination/   # Extract to a specific directory
tar --exclude='*.log' -czvf archive.tar.gz myfolder/   # Exclude specific file patterns
```
**Flags explained**:
```sh
`c` = create, 
`x` = extract, 
`t` = list/test, 
`v` = verbose (show progress), 
`f` = specify filename (always comes last before the filename), 
`z` = gzip compression, 
`j` = bzip2 compression, 
`C` = change to directory before extracting.
```

**Sample Output**:
```bash
$ tar -czvf backup.tar.gz /home/ubuntu/project/
tar: Removing leading `/' from member names
home/ubuntu/project/
home/ubuntu/project/app.py
home/ubuntu/project/config.yml
```
## 8.4 zip / unzip Commands

```bash
sudo apt install zip unzip -y     # Install if not present (Ubuntu)
sudo dnf install zip unzip -y      # Install on RHEL/Amazon Linux

zip archive.zip file1.txt file2.txt      # Zip specific files
zip -r archive.zip myfolder/               # Zip a directory recursively
unzip archive.zip                           # Extract a zip file
unzip archive.zip -d /destination/           # Extract to a specific location
unzip -l archive.zip                          # List contents without extracting
zip -r -x "*.log" archive.zip myfolder/        # Zip while excluding .log files
```
## 8.5 tar vs zip — Comparison

|Feature |tar (+gzip) |zip |
|---|---|---|
|Native to |Linux/Unix |Windows (but works everywhere) |
|Compression |Separate step (gzip/bzip2) |Built-in |
|Preserves Linux permissions |Yes |Limited |
|Best for |Linux-to-Linux transfers, backups |Cross-platform sharing |
|Typical extension |.tar.gz, .tar.bz2, .tgz |.zip |

## 8.6 Real-World Example

Before deploying your application to a new EC2 instance, you might package it:

```bash
tar --exclude='node_modules' --exclude='.git' -czvf myapp.tar.gz ./myapp/
scp -i key.pem myapp.tar.gz ubuntu@54.x.x.x:/home/ubuntu/
ssh -i key.pem ubuntu@54.x.x.x "tar -xzvf myapp.tar.gz"
```
## 8.7 Best Practices

- Always exclude unnecessary files (node_modules, .git, __pycache__) before archiving to keep transfer sizes small.
- Use .tar.gz for Linux server-to-server transfers (better permission preservation than zip).
- Test archives after creation with tar -tvf before relying on them for backups.
- Name archives with dates for backup rotation clarity, e.g., backup_2026-07-01.tar.gz.

## 8.8 Common Mistakes

- Forgetting the f flag position (must come immediately before the filename) — a classic tar syntax error.
- Using zip for Linux permission-sensitive backups, then being surprised permissions aren't preserved on restore.
- Not using --exclude and accidentally archiving huge unnecessary directories like node_modules or .git, bloating the archive.


## 8.9 Interview Questions

1. What's the difference between tar -czvf and tar -cjvf?
2. How would you extract a .tar.gz file to a specific directory?
3. When would you choose zip over tar?
4. How do you list the contents of an archive without extracting it?
5. How would you exclude the .git folder while archiving a project?


## 8.10 Scenario-Based Question

**Scenario**: You need to back up /var/www/html every night and store the last 7 days of backups, deleting anything older, using a cron job. Sketch the script and cron entry.

> (Approach: A bash script using tar -czvf /backups/html_$(date +%F).tar.gz /var/www/html followed by find /backups -name "html_*.tar.gz" -mtime +7 -delete, scheduled via crontab -e at 0 2 * * *.)

## 8.11 Troubleshooting

**Issue**: tar: Error is not recoverable: exiting now.

**Fix**: Usually caused by insufficient disk space or a corrupted archive; check df -h first.

**Issue**: Extracted files have wrong ownership.

**Fix**: Use sudo tar -xzvf ...  if extracting system files, or chown -R afterward.


## 8.12 Cheat Sheet

```bash
tar -czvf out.tar.gz dir/     # Create gzip archive
tar -xzvf out.tar.gz           # Extract gzip archive
tar -tvf out.tar.gz            # List contents
zip -r out.zip dir/             # Create zip archive
unzip out.zip                    # Extract zip
unzip -l out.zip                  # List zip contents
```
## 8.13 Summary

Archiving with tar and zip is a small but essential skill for packaging code, creating backups, and transferring applications efficiently. tar.gz is the Linux-native standard you'll use constantly; zip is your cross-platform fallback.
