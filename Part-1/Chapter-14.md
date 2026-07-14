# CHAPTER 14: PACKAGE MANAGEMENT — apt, snap, REPOSITORY MANAGEMENT

## 14.1 Theory

A package manager automates installing, updating, configuring, and removing software, along with automatically resolving dependencies (other software your program needs to run). Debian-based distros (Ubuntu) use APT (Advanced Package Tool) with .deb packages; Red Hat-based distros (RHEL, CentOS, Amazon Linux, Fedora) use DNF/YUM with .rpm packages. Snap is a newer, universal packaging format that works across distros.

## 14.2 Why It Exists

Before package managers, installing software meant manually downloading source code, compiling it, and manually tracking every dependency — a nightmare, especially at scale. Package managers turn sudo apt install nginx into a single command that fetches the correct version, all its dependencies, and configures it correctly, from a trusted, centrally-maintained repository.

## 14.3 APT — Debian/Ubuntu Package Management

```bash
sudo apt update                       # Refresh the list of available packages (does NOT install anything)
sudo apt upgrade                        # Upgrade all installed packages to their latest versions
sudo apt update && sudo apt upgrade -y    # Combined — the -y auto-confirms prompts
sudo apt install nginx                      # Install a package
sudo apt install nginx=1.18.0-0ubuntu1        # Install a SPECIFIC version
sudo apt remove nginx                            # Remove a package but keep its config files
sudo apt purge nginx                               # Remove a package AND its config files
sudo apt autoremove                                  # Remove packages no longer needed (orphaned dependencies)
apt list --installed                                    # List all installed packages
apt search nginx                                          # Search for a package by name/keyword
apt show nginx                                              # Show detailed info about a package
sudo apt clean                                                # Clear the local package cache to free disk space
```
Sample Output of sudo apt update:
```sh
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]
Fetched 119 kB in 1s (95.2 kB/s)
Reading package lists... Done
Building dependency tree... Done
5 packages can be upgraded. Run 'apt list --upgradable' to see them.
```
## 14.4 DNF/YUM — Red Hat/CentOS/Amazon Linux Package Management


```bash
sudo dnf update                    # Update all packages (yum update on older systems)
sudo dnf install nginx                # Install a package
sudo dnf remove nginx                    # Remove a package
dnf list installed                          # List installed packages
dnf search nginx                              # Search for a package
dnf info nginx                                  # Show package details
sudo dnf clean all                                # Clear cache
```
## 14.5 apt vs dnf/yum — Direct Command Comparison

|Task |Ubuntu/Debian (apt) |RHEL/CentOS/Amazon Linux (dnf/yum) |
|---|---|---|
|Refresh package list |`apt update` |(`dnf` does this automatically) |
|Upgrade all packages |`apt upgrade` |`dnf update` |
|Install a package |`apt install nginx` |`dnf install nginx` |
|Remove a package |`apt remove nginx` |`dnf remove nginx` |
|Search for a package |`apt search nginx` |`dnf search nginx` |
|List installed |`apt list --installed` |`dnf list installed` |
|Package file format |`.deb` |`.rpm` |

## 14.6 snap — Universal Packages

```bash
sudo snap install code --classic      # Install a snap package (--classic for apps needing broader system access)
snap list                               # List installed snaps
sudo snap refresh                         # Update all snaps
sudo snap remove code                        # Remove a snap
snap find docker                               # Search for a snap package
```
apt vs snap — When to Use Which:

| |apt |snap |
|---|---|---|
| |SpeedFast, lightweight |Slower to launch (sandboxed) |
|Isolation |Shares system libraries |Fully sandboxed, bundles its own dependencies |
|Auto-updates |Manual (apt upgrade) |Automatic by default |
|Best for |Core system tools, servers |Desktop apps, cross-distro tools |

## 14.7 Repository Management

A repository ("repo") is a server hosting a collection of packages. Your package manager needs to know which repos to trust and check.

```bash
cat /etc/apt/sources.list                        # View configured repositories (Ubuntu)
ls /etc/apt/sources.list.d/                         # Additional repo config files (often one per third-party source)

# Adding a third-party repository (real example: adding Docker's official repo)
sudo apt install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce -y
```
On RHEL/CentOS/Amazon Linux, repos are managed similarly via .repo files in /etc/yum.repos.d/:

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce -y
```
## 14.8 Architecture Diagram — How Package Installation Works
```sh
   sudo apt install nginx
             │
             ▼
   1. Check /etc/apt/sources.list — WHICH repos are trusted?
             │
             ▼
   2. Query repo servers: does 'nginx' exist, what version, what dependencies?
             │
             ▼
   3. Resolve dependency tree (nginx needs libpcre3, zlib1g, etc.)
             │
             ▼
   4. Download all .deb files to local cache (/var/cache/apt/archives)
             │
             ▼
   5. Verify package signatures (security check against tampering)
             │
             ▼
   6. Install/unpack files to correct system locations (/usr/sbin/nginx, /etc/nginx/, etc.)
             │
             ▼
   7. Run post-install scripts (e.g., create 'www-data' user, enable systemd service)
```
## 14.9 Best Practices


- Always run apt update before apt install on a fresh server — an outdated package index can cause "package not found" errors even for real packages.
- Pin critical production package versions instead of blindly running apt upgrade, to avoid unexpected breaking changes.
- Only add third-party repositories from trusted, official sources (verify GPG keys) — this is a real attack vector if done carelessly.
- Regularly run apt autoremove and apt clean to keep servers tidy and free disk space.
- Prefer apt/dnf for server software and reserve snap mainly for desktop-style or cross-distro tools.


## 14.10 Common Mistakes


- Confusing apt update (refresh package LIST) with apt upgrade (actually upgrade INSTALLED packages) — one of the most common beginner mix-ups.
- Running apt install without sudo and getting confusing "permission denied" errors.
- Adding untrusted third-party repositories without verifying their GPG signing key, exposing the system to supply-chain attacks.
- Forgetting that on Amazon Linux 2023, yum is deprecated in favor of dnf (though often still aliased for compatibility).


## 14.11 Interview Questions


1. What's the difference between apt update and apt upgrade?
2. How does APT resolve package dependencies automatically?
3. What's the difference between apt remove and apt purge?
4. How would you add a third-party repository (e.g., Docker's official repo) to an Ubuntu server?
5. What's the difference between .deb and .rpm package formats, and which distros use each?
6. When would you choose snap over apt, if ever?
7. How would you check what version of a package is currently installed?


## 14.12 Scenario-Based Question

**Scenario**: You need to install Docker on a fresh Ubuntu 22.04 EC2 instance, but the version in Ubuntu's default repository is outdated. How do you get the latest official version?

> (Approach: Add Docker's official APT repository (as shown in section 14.7) with the correct GPG key, run apt update to refresh the package index now including Docker's repo, then apt install docker-ce docker-ce-cli containerd.io -y to get the latest official build rather than Ubuntu's older bundled version.)

## 14.13 Troubleshooting


Issue: E: Unable to locate package nginx.

Fix: Run sudo apt update first — the local package index is likely stale or missing that repo.

Issue: E: Could not get lock /var/lib/dpkg/lock-frontend.

Fix: Another apt/dpkg process is running (often an automatic unattended-upgrade); wait for it to finish, or check ps aux | grep apt and only kill it if you're certain it's stuck.

Issue: GPG signature errors when adding a new repo.

Fix: Ensure the repository's public key was correctly imported/dearmored to the expected keyring location before running apt update.


## 14.14 Cheat Sheet

```bash
# APT (Ubuntu/Debian)
sudo apt update                  # Refresh package list
sudo apt upgrade -y                # Upgrade installed packages
sudo apt install <pkg> -y            # Install package
sudo apt remove <pkg>                  # Remove package (keep config)
sudo apt purge <pkg>                     # Remove package + config
sudo apt autoremove                        # Clean unused dependencies
apt search <pkg>                              # Search for package
apt list --installed                             # List installed packages

# DNF/YUM (RHEL/CentOS/Amazon Linux)
sudo dnf update                  # Update packages
sudo dnf install <pkg> -y          # Install package
sudo dnf remove <pkg>                # Remove package
dnf search <pkg>                        # Search for package
dnf list installed                          # List installed packages

# SNAP (universal)
sudo snap install <pkg>          # Install snap
snap list                          # List installed snaps
sudo snap remove <pkg>               # Remove snap
```
## 14.15 Summary

Package managers (apt for Debian/Ubuntu, dnf/yum for Red Hat family, snap for universal apps) are how you install and maintain every piece of software on a Linux server — from Nginx to Docker to Jenkins. Understanding repository management, especially adding trusted third-party repos, is exactly how you'll install modern DevOps tooling (Docker, Terraform, kubectl) on any fresh cloud instance.
