# CHAPTER 11: curl AND wget

## 11.1 Theory

curl and wget are command-line tools for transferring data to/from a server using protocols like HTTP, HTTPS, and FTP. They are essential for testing APIs, downloading files/scripts, and health-checking web services directly from the terminal — no browser needed.

## 11.2 Why It Exists

As a DevOps engineer, you constantly need to verify "is my web service actually responding correctly?" without opening a browser (especially on headless servers with no GUI at all). curl and wget let you script these checks, download installation files during automated setups, and interact with REST APIs (like the AWS CLI does under the hood, and like you'll do with Jenkins/GitHub webhooks).

## 11.3 curl — Client URL

```bash
curl https://example.com                       # Fetch and print HTML/response body to terminal
curl -I https://example.com                      # Fetch ONLY the HTTP headers (no body) - great for quick health checks
curl -o output.html https://example.com            # Save output to a file
curl -O https://example.com/file.zip                 # Save with the REMOTE file's original name
curl -L https://short.url/redirect                     # Follow redirects (curl doesn't follow by default)
curl -X POST https://api.example.com/users               # Send a POST request
curl -X POST -H "Content-Type: application/json" -d '{"name":"Ambika"}' https://api.example.com/users
                                                              # POST with JSON body and custom header
curl -u username:password https://api.example.com            # Basic authentication
curl -H "Authorization: Bearer <token>" https://api.example.com  # Token-based auth (common for REST APIs)
curl -s https://example.com                                        # Silent mode (no progress bar) — great for scripts
curl -w "%{http_code}\n" -o /dev/null -s https://example.com          # Print ONLY the HTTP status code
```
Sample Output of curl -I:
```bash
$ curl -I https://example.com
HTTP/2 200
content-type: text/html; charset=UTF-8
content-length: 1256
date: Wed, 01 Jul 2026 10:00:00 GMT
```
A 200 status confirms the site is healthy. This exact one-liner (curl -I) is often used in monitoring scripts and Jenkins pipeline health checks.

## 11.4 wget — Web Get

wget is optimized specifically for downloading files, with better support for resuming interrupted downloads and recursive downloads.

```bash
wget https://example.com/file.zip                # Download a file
wget -O myfile.zip https://example.com/file.zip     # Download and save with a custom name
wget -c https://example.com/largefile.iso            # Resume a partially downloaded file (continue)
wget -q https://example.com/file.zip                   # Quiet mode (no progress output)
wget -b https://example.com/largefile.iso                # Download in the background
wget --limit-rate=200k https://example.com/file.zip        # Throttle download speed
wget -r -np https://example.com/docs/                        # Recursively download a whole directory of files
```
11.5 curl vs wget — Comparison

|Feature |curl |wget |
|---|---|---|
|Primary purpose |API testing, sending data (GET/POST/PUT/DELETE) |Downloading files |
|Resume interrupted downloads |Limited |Excellent (-c) |
|Recursive downloads |No |Yes (-r) |
|Output to terminal by default |Yes |No (saves to file by default) |
|Best for |Testing REST APIs, scripting HTTP checks |Downloading installers, ISOs, files in bulk |

## 11.6 Real-World Example

Health-checking your deployed EC2 web server from your local machine or a monitoring script:

```bash
curl -s -o /dev/null -w "%{http_code}" http://54.210.15.22
# Returns just "200" if healthy — perfect for scripting a simple uptime check
```
Downloading and installing a tool during server setup (this is literally what many installation guides tell you to do):

```bash
curl -fsSL https://get.docker.com | sh          # Common pattern: download & pipe install script directly to shell
wget https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip
```
## 11.7 Best Practices


- Always inspect install scripts before piping them into sh (curl ... | sh) — understand what you're running on your server, especially in production.
- Use curl -I or curl -s -o /dev/null -w "%{http_code}" for lightweight health checks in scripts rather than downloading full page content unnecessarily.
- Use wget -c for large file downloads over unreliable connections so you don't restart from zero on failure.
- Always use HTTPS URLs when possible for both tools to ensure encrypted transfer.


## 11.8 Common Mistakes


- Forgetting -L in curl when a URL redirects, leading to unexpectedly empty output.
- Using curl when you actually just want to download and save a file — -O is easy to forget, resulting in the file content being dumped into your terminal instead.
- Not quoting URLs containing special characters (&, ?) in bash, causing the shell to misinterpret the command.
- Blindly piping curl output into sh without checking the source's trustworthiness.


## 11.9 Interview Questions


1. What's the core difference between curl and wget?
2. How would you check only the HTTP status code of a URL using curl, without printing the whole page?
3. How do you send a POST request with a JSON body using curl?
4. How would you resume an interrupted large file download?
5. Why should you be cautious about curl <script-url> | sh commands?


## 11.10 Scenario-Based Question

**Scenario**: You need to write a simple bash health-check script that curls your website every minute and logs an alert if the HTTP status isn't 200. Describe your approach.

> (Approach: Use curl -s -o /dev/null -w "%{http_code}" inside a bash while true loop or a cron job every minute, compare the returned code to "200", and echo a timestamped alert to a log file or send a notification if it doesn't match — this is a simplified precursor to what CloudWatch/monitoring tools do automatically, covered in later Parts.)

## 11.11 Troubleshooting


- *Issue*: curl: (6) Could not resolve host.

Fix: DNS issue — check /etc/resolv.conf, verify internet connectivity with ping 8.8.8.8, check for typos in the URL.

- *Issue*: curl: (7) Failed to connect... Connection refused.

Fix: The service isn't listening on that port, or a firewall/Security Group is blocking it — check with ss -tulnp on the server side.


## 11.12 Cheat Sheet

```bash
curl https://url                     # GET request, print body
curl -I https://url                    # Headers only
curl -O https://url/file                 # Download, keep filename
curl -o name.zip https://url/file          # Download, custom name
curl -X POST -d '{"k":"v"}' -H "Content-Type: application/json" url  # POST JSON
wget https://url/file                        # Download file
wget -c https://url/file                      # Resume download
wget -O name.zip https://url/file               # Download, custom name
```
## 11.13 Summary

curl and wget are your terminal's window into the web — curl for testing and interacting with APIs, wget for reliable file downloads. Both appear constantly in installation scripts, CI/CD pipelines, and health-check automation, making them essential daily tools.
