CHAPTER 7: NETWORKING COMMANDS, SSH, SCP, rsync

7.1 Theory

Networking commands let you inspect connectivity, diagnose issues, and transfer files/data securely between machines. SSH (Secure Shell) is the single most important protocol in this chapter — it's how you'll access every EC2 instance you ever launch.

7.2 Why It Exists

Before SSH, protocols like Telnet sent data (including passwords!) in plain text across the network — anyone snooping on the network could read everything. SSH encrypts the entire session using public-key cryptography, making remote administration secure. This is precisely the mechanism behind the .pem key file AWS gives you when you launch an EC2 instance.

7.3 Core Networking Commands

bashping google.com               # Test connectivity/latency to a host
ping -c 4 8.8.8.8               # Send exactly 4 pings then stop
traceroute google.com           # Show the network path (hops) to a destination
hostname                         # Show this machine's hostname
hostname -I                      # Show this machine's IP address(es)
ip addr show                     # Modern way to view network interfaces & IPs
ip a                              # Shorthand for the above
ifconfig                          # Older/legacy way to view interfaces (may need net-tools installed)
ip route                          # Show routing table
cat /etc/hosts                    # Local hostname-to-IP mappings
cat /etc/resolv.conf                # DNS servers this machine uses
dig google.com                     # Detailed DNS lookup
nslookup google.com                 # Simpler DNS lookup
telnet <host> <port>                 # Test if a specific port is open (basic)
nc -zv <host> <port>                  # Netcat: test port connectivity (better than telnet)

7.4 SSH — Secure Shell

Basic connection:

bashssh username@hostname_or_ip
ssh ubuntu@54.210.15.22
ssh -i mykey.pem ubuntu@54.210.15.22       # Connect using a private key (AWS EC2 style)
ssh -p 2222 ubuntu@54.210.15.22             # Connect on a non-default port

Generating SSH key pairs (for passwordless login):

bashssh-keygen -t rsa -b 4096 -C "ambika@example.com"
# Creates: ~/.ssh/id_rsa (private key - NEVER share this)
#          ~/.ssh/id_rsa.pub (public key - safe to share/upload)

ssh-copy-id ubuntu@54.210.15.22    # Copy your public key to a remote server for passwordless login

Architecture Diagram — SSH Key Authentication Flow:

  YOUR LAPTOP                              EC2 INSTANCE
  ┌─────────────┐                        ┌─────────────────┐
  │ id_rsa       │                        │ ~/.ssh/          │
  │ (PRIVATE KEY)│──── SSH connect ──────►│ authorized_keys  │
  │ id_rsa.pub   │                        │ (contains your   │
  └─────────────┘                        │  PUBLIC key)      │
                                          └─────────────────┘
  1. Server sends a challenge encrypted with your public key
  2. Only YOUR private key can decrypt it
  3. You prove possession of the private key WITHOUT ever sending it
  4. Server grants access

SSH Config File — for convenience, edit ~/.ssh/config:

Host mystaging
    HostName 54.210.15.22
    User ubuntu
    IdentityFile ~/.ssh/mykey.pem
    Port 22

Now you can simply run: ssh mystaging instead of typing the full command every time.

7.5 SCP — Secure Copy

scp copies files between local and remote machines over SSH.

bash# Copy a LOCAL file TO a remote server
scp -i mykey.pem myfile.txt ubuntu@54.210.15.22:/home/ubuntu/

# Copy a REMOTE file TO your local machine
scp -i mykey.pem ubuntu@54.210.15.22:/home/ubuntu/logs.txt ./

# Copy an ENTIRE directory recursively
scp -i mykey.pem -r ./myproject ubuntu@54.210.15.22:/home/ubuntu/

7.6 rsync — Efficient File Synchronization

rsync is smarter than scp — it only transfers the differences between source and destination, making it much faster for repeated syncs or large files.

bashrsync -avz ./myproject/ ubuntu@54.210.15.22:/home/ubuntu/myproject/
# -a = archive mode (preserves permissions, timestamps, symlinks, recursive)
# -v = verbose (show what's being copied)
# -z = compress data during transfer

rsync -avz --delete ./local/ ubuntu@remote:/remote/path/
# --delete = remove files in destination that no longer exist in source (mirror mode)

rsync -avzP ./bigfile.zip ubuntu@remote:/home/ubuntu/
# -P = show progress bar AND allow resuming interrupted transfers

rsync -avz -e "ssh -i mykey.pem" ./project/ ubuntu@remote:/home/ubuntu/project/
# -e = specify a custom SSH command (needed when using a specific key file)

scp vs rsync — When to Use Which:

FeaturescprsyncSpeed on repeat transfersSlow (copies everything again)Fast (copies only changes)Resume interrupted transferNoYes (-P flag)Mirror/delete extra filesNoYes (--delete)Simplicity for one-off copySimplerSlightly more flagsBest forQuick single file copyBackups, deployments, large directories

7.7 Best Practices


Always use SSH key-based authentication instead of passwords — disable password auth entirely on production servers (PasswordAuthentication no in /etc/ssh/sshd_config).
Store .pem files securely with chmod 400 and never commit them to Git.
Prefer rsync over scp for deployments and backups — it's faster and idempotent.
Use an SSH config file (~/.ssh/config) to manage multiple servers cleanly instead of memorizing IPs and key paths.


7.8 Common Mistakes


Losing your .pem file — AWS cannot regenerate it; you'd need to detach the EBS volume and manually add a new key (advanced recovery process).
Forgetting -r when using scp to copy a directory (fails with "not a regular file").
Using scp repeatedly for large ongoing syncs instead of rsync, wasting massive time/bandwidth.
Not restricting Security Group rules for port 22 to your own IP, leaving SSH open to the entire internet (0.0.0.0/0) — a major AWS security mistake covered further in Part 2.


7.9 Interview Questions


Explain how SSH key-based authentication works, step by step.
What's the difference between scp and rsync?
Why would you use rsync -avz --delete, and what risk does it carry?
How do you connect to an EC2 instance using a .pem key file?
What does chmod 400 on a .pem file accomplish, and why is it required?
How would you test if a specific port is open on a remote server?


7.10 Scenario-Based Question

Scenario: You need to deploy an updated version of your web app to 3 different EC2 instances every time you push code. Manually running scp each time is slow and error-prone. Propose a better approach using tools from this chapter (before we cover full CI/CD in later Parts).

(Approach: Write a bash script looping over the 3 server IPs, using rsync -avz -e "ssh -i key.pem" for each, ideally combined with SSH config aliases; mention this is exactly the kind of problem Ansible — covered in a later Part — was built to solve at scale.)

7.11 Troubleshooting


Issue: Permission denied (publickey).
Fix: Verify correct key (-i path/to/key.pem), correct username (ubuntu for Ubuntu AMIs, ec2-user for Amazon Linux/RHEL AMIs), and key file permissions (chmod 400).
Issue: Connection timed out.
Fix: Check the EC2 Security Group allows inbound traffic on port 22 from your IP; check the instance is running; check you're using the correct public IP (it changes on stop/start unless you use an Elastic IP).


7.12 Cheat Sheet

bashssh -i key.pem user@ip                     # Connect via SSH
ssh-keygen -t rsa -b 4096                   # Generate key pair
scp -i key.pem file user@ip:/path/          # Copy file to remote
scp -i key.pem -r dir user@ip:/path/         # Copy directory to remote
rsync -avz src/ user@ip:/dest/               # Sync directory
rsync -avzP file user@ip:/dest/               # Sync with progress + resume
ping -c 4 host                               # Test connectivity
ip a                                          # Show IP addresses
nc -zv host port                              # Test port connectivity

7.13 Summary

SSH is the gateway to every remote Linux server you'll ever manage, and scp/rsync are how you move files across that gateway. Master SSH key authentication deeply — it's tested in nearly every practical DevOps interview and it's literally how you'll access AWS EC2 instances from Day 1 of any real job.
