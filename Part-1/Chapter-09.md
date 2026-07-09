# CHAPTER 9: TEXT PROCESSING — grep, awk, sed, find, locate

## 9.1 Theory

Text processing tools let you search, filter, transform, and locate data — an essential skill since most of what a DevOps engineer does is read and manipulate logs, config files, and command output. These tools are designed to be piped together (|), following the Unix philosophy: "do one thing well, and combine simple tools to solve complex problems."

## 9.2 grep — Search Text Using Patterns

```bash
grep "error" logfile.txt                # Find lines containing "error"
grep -i "error" logfile.txt               # Case-insensitive search
grep -v "error" logfile.txt                # Invert match — show lines WITHOUT "error"
grep -r "TODO" ./src/                       # Recursive search through a directory
grep -n "error" logfile.txt                  # Show line numbers
grep -c "error" logfile.txt                   # Count matching lines
grep -E "error|fail|critical" logfile.txt      # Extended regex — match multiple patterns (OR)
grep -A 3 "error" logfile.txt                   # Show 3 lines AFTER each match
grep -B 3 "error" logfile.txt                    # Show 3 lines BEFORE each match
grep -w "cat" file.txt                            # Match whole word only (not "concatenate")
```
**Real-World Example**:

```bash
cat /var/log/nginx/error.log | grep -i "connection refused"
journalctl -u myapp | grep -i "exception" | tail -20
```
## 9.3 awk — Pattern Scanning and Text Processing Language

`awk` is a full text-processing programming language, most commonly used to extract and process columns of data.

```bash
awk '{print $1}' file.txt                 # Print the first column (space-delimited by default)
awk '{print $1, $3}' file.txt               # Print columns 1 and 3
awk -F: '{print $1}' /etc/passwd             # Use ':' as the delimiter (list usernames)
awk '{print NR, $0}' file.txt                 # Print line number + entire line
awk '/error/ {print}' logfile.txt               # Print lines matching "error" (like grep)
awk '{sum += $2} END {print sum}' data.txt        # Sum column 2 across all lines
awk '$3 > 100 {print $1}' data.txt                 # Print column 1 where column 3 > 100
ps aux | awk '{print $2, $11}'                       # Print PID and command from ps output
```
**Real-World Example**: Find which process is using the most memory:

```bash
ps aux | awk '{print $4, $11}' | sort -rn | head -5
```
## 9.4 sed — Stream Editor (Find and Replace)

`sed` processes text line-by-line, most commonly for search-and-replace operations, without opening a file in an editor.

```bash
sed 's/old/new/' file.txt                  # Replace FIRST occurrence per line (prints to screen only)
sed 's/old/new/g' file.txt                   # Replace ALL occurrences per line
sed -i 's/old/new/g' file.txt                 # -i = edit the file IN PLACE (actually saves changes)
sed -i.bak 's/old/new/g' file.txt              # Edit in place but keep a .bak backup first
sed -n '5,10p' file.txt                          # Print only lines 5 through 10
sed '/^#/d' file.txt                              # Delete all lines starting with # (comments)
sed -i '/^$/d' file.txt                            # Delete all blank lines, in place
```
**Real-World Example**: Update a config value across all servers:

```bash
sed -i 's/max_connections=100/max_connections=500/' /etc/myapp/config.conf
```
## 9.5 find — Locate Files by Criteria

```bash
find /var/log -name "*.log"                   # Find files by name pattern
find / -type f -name "*.conf" 2>/dev/null       # Find only files (not directories), suppress permission errors
find / -type d -name "logs"                      # Find only directories
find . -mtime -7                                   # Files modified in the last 7 days
find . -mtime +30                                    # Files modified MORE than 30 days ago
find . -size +100M                                     # Files larger than 100MB
find /tmp -type f -mtime +7 -delete                     # Find AND delete files older than 7 days
find . -name "*.sh" -exec chmod +x {} \;                 # Find .sh files and chmod each one
find / -user ambika                                        # Find all files owned by a specific user
find / -perm 777                                             # Find files with dangerous 777 permissions
```
## 9.6 locate — Fast File Search (Using a Pre-Built Index)

```bash
sudo apt install mlocate -y      # Install (Ubuntu)
sudo updatedb                     # Manually refresh the search index (runs automatically via cron normally)
locate nginx.conf                  # Instantly search the index for matching files
locate -i readme                    # Case-insensitive search
```
**find vs locate — Comparison**:

|Feature |find |locate |
|---|---|---|
|Speed |Slower (searches live filesystem) |Very fast (searches pre-built index) |
|Accuracy |Always current |Can be stale if updatedb hasn't run recently |
|Search by criteria (size, date, permission) |Yes, very powerful |No, name-only search |
|Best for |Precise, complex searches |Quick "where is this file" lookups |

## 9.7 Combining Tools — The Real Power of Linux (Pipes)

```bash
# Find the top 5 IPs hitting your Nginx server (common security/traffic analysis task)
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -5

# Count how many ERROR lines occurred today in an app log
grep "ERROR" app.log | grep "$(date +%Y-%m-%d)" | wc -l

# Find all failed SSH login attempts
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn
```
## 9.8 Architecture Diagram — The Unix Pipe Philosophy
```sh
   cat access.log  │  awk '{print $1}'  │  sort  │  uniq -c  │  sort -rn  │  head -5
   ─────────────   │  ─────────────────  │ ────── │ ───────── │ ────────── │ ────────
   Read raw log  ──►  Extract IP column ──► Sort ──► Count dups ──► Sort by count ──► Top 5
```
Each tool does ONE job well; the pipe | passes output from one command as input to the next, letting you build powerful one-liners from simple building blocks.

## 9.9 Best Practices

- Always test sed -i and find -delete on a COPY of data first, or use -n/dry-run equivalents — both make irreversible changes.
- Use grep -r combined with --include="*.py" to search only specific file types in large codebases.
- Learn to read awk column-based output — it's the fastest way to extract data from ps, df, netstat, and log files.
- Chain tools with pipes rather than writing complex one-off scripts for simple text tasks.

## 9.10 Common Mistakes

- Running sed -i without a backup (-i.bak) and realizing too late the replacement was wrong.
- Forgetting 2>/dev/null when running find / as a non-root user, resulting in a screen full of "Permission denied" noise.
- Using find ... -delete without first running the same find command WITHOUT -delete to preview exactly what will be removed.
- Confusing grep -v (invert match) with simply not understanding why "matching" lines are missing from output.

## 9.11 Interview Questions

1. What's the difference between find and locate?
2. Write a command to find all .log files larger than 50MB modified in the last 3 days.
3. How would you replace all occurrences of "8080" with "9090" in a config file, saving the change?
4. Write an awk command to print the 2nd and 4th columns of a space-separated file.
5. How would you find the top 10 IP addresses accessing your web server from an access log?
6. What does grep -v do?
7. Why might you prefer sed -i.bak over plain sed -i in production?


## 9.12 Scenario-Based Question

**Scenario**: Your Nginx access log has grown to 2GB and you suspect a bot is spamming a specific endpoint. Using only the tools in this chapter, find the top 10 IPs and the specific URL they're hitting the most.

> (Approach: awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10 for top IPs, then grep "<suspect-IP>" access.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -5 to find which URL that IP hits most.)

## 9.13 Troubleshooting


**Issue**: sed command runs but the file is unchanged.

**Fix**: You likely forgot the -i flag — without it, sed only prints to the terminal, it doesn't save changes.

**Issue**: find returns "Permission denied" spam.

**Fix**: Append 2>/dev/null to suppress stderr, or run with sudo if you need to search protected directories.


## 9.14 Cheat Sheet

```bash
grep -i "text" file          # Case-insensitive search
grep -r "text" dir/            # Recursive search
grep -v "text" file             # Invert match
awk '{print $1}' file            # Print column 1
awk -F: '{print $1}' file         # Custom delimiter
sed 's/old/new/g' file             # Replace all (preview only)
sed -i 's/old/new/g' file           # Replace all (save to file)
find . -name "*.log"                 # Find by name
find . -mtime -7                      # Modified in last 7 days
find . -size +100M                     # Larger than 100MB
locate filename                          # Fast indexed search
sudo updatedb                             # Refresh locate's index
```
## 9.15 Summary

grep, awk, sed, find, and locate are the workhorses of daily Linux administration — you'll use them to hunt down errors in logs, bulk-edit configuration files, and locate files across huge servers. Combined with pipes, these five tools alone can solve the majority of real-world text-processing problems you'll face in DevOps.
