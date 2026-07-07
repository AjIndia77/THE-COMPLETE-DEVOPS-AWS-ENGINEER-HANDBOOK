CHAPTER 13: BASH SCRIPTING & ENVIRONMENT VARIABLES

13.1 Theory

A bash script is a plain text file containing a sequence of Linux commands, executed top to bottom by the bash shell — essentially "automating what you'd otherwise type manually." Environment variables are named values available to the shell and all processes it launches, used to configure behavior (like where to find executable programs, via PATH).

13.2 Why It Exists

Manually typing the same 10 commands every time you deploy an app is slow and error-prone. A bash script turns that into one command: ./deploy.sh. This is literally the foundation that CI/CD pipelines (Jenkins, GitHub Actions) are built on — Part 3 and 4 of this book will show you that even "advanced" CI/CD pipelines are, under the hood, sequences of bash commands.

13.3 Your First Bash Script

bash#!/bin/bash
 This is a comment. The line above is called a "shebang" — it tells
 the OS which interpreter to use to run this script.

echo "Starting deployment..."
cd /home/ubuntu/myapp
git pull origin main
sudo systemctl restart myapp
echo "Deployment complete!"

Running a script:

bashchmod +x deploy.sh      # Make it executable (see Chapter 4)
./deploy.sh               # Run it (./ means "in the current directory")
bash deploy.sh              # Alternative way to run without chmod +x

13.4 Variables

bash#!/bin/bash
NAME="Ambika"
APP_DIR="/home/ubuntu/myapp"
echo "Hello, $NAME"           # Use $ to READ a variable's value
echo "App is at ${APP_DIR}"    # Curly braces {} are safer, especially next to other text

13.5 User Input & Command-Line Arguments

bash#!/bin/bash
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME"

# Command-line arguments passed when running the script:
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"

bash./script.sh apple banana
# $1 = apple, $2 = banana, $# = 2

13.6 Conditionals (if / else)

bash#!/bin/bash
read -p "Enter a number: " NUM

if [ $NUM -gt 100 ]; then
    echo "Number is greater than 100"
elif [ $NUM -eq 100 ]; then
    echo "Number is exactly 100"
else
    echo "Number is less than 100"
fi

Common comparison operators:

-eq   equal              -ne  not equal
-gt   greater than        -lt  less than
-ge   greater or equal      -le  less or equal
-z    string is empty         -n   string is NOT empty
-f    file exists                -d   directory exists

Real-World Example — Service Health Check Script:

bash#!/bin/bash
if systemctl is-active --quiet nginx; then
    echo "Nginx is running"
else
    echo "Nginx is DOWN — restarting..."
    sudo systemctl start nginx
fi

13.7 Loops (for / while)

bash#!/bin/bash
# for loop — iterate over a list
for SERVER in server1 server2 server3; do
    echo "Deploying to $SERVER..."
done

# for loop — iterate over a range of numbers
for i in {1..5}; do
    echo "Attempt number $i"
done

# while loop — repeat while a condition is true
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count is $COUNT"
    COUNT=$((COUNT + 1))
done

13.8 Functions

bash#!/bin/bash
deploy_app() {
    echo "Deploying application..."
    cd /home/ubuntu/myapp
    git pull
    systemctl restart myapp
    echo "Done."
}

deploy_app    # Call the function

13.9 A Realistic, Complete DevOps Script

This mirrors exactly the kind of automation you built in your EC2 project (setup.sh):

bash#!/bin/bash
# Automated Nginx deployment script

set -e   # Exit immediately if any command fails (CRITICAL for production scripts)

echo "=== Updating system packages ==="
sudo apt update -y

echo "=== Installing Nginx ==="
sudo apt install nginx -y

echo "=== Deploying custom HTML page ==="
sudo cp index.html /var/www/html/index.html
sudo chown www-data:www-data /var/www/html/index.html

echo "=== Starting and enabling Nginx ==="
sudo systemctl enable --now nginx

echo "=== Verifying deployment ==="
if curl -s -o /dev/null -w "%{http_code}" http://localhost | grep -q "200"; then
    echo "SUCCESS: Nginx is serving the page correctly."
else
    echo "ERROR: Deployment verification failed!"
    exit 1
fi

13.10 Environment Variables

Viewing and setting variables:

bashenv                       # List all environment variables
printenv                   # Same as above
echo $HOME                   # Print a specific variable
echo $PATH                     # Print the PATH variable (where bash looks for commands)
export MY_VAR="hello"             # Set a NEW environment variable for this session
unset MY_VAR                        # Remove a variable

Common Built-in Environment Variables:

VariableMeaning$HOMECurrent user's home directory$PATHColon-separated list of directories bash searches for executable commands$USERCurrent logged-in username$PWDCurrent working directory$SHELLPath to the current shell (e.g., /bin/bash)$HOSTNAMEMachine's hostname

Making variables permanent — session-only export disappears when you log out. To persist:

bash# Add to ~/.bashrc (per-user) or /etc/environment (system-wide)
echo 'export JAVA_HOME=/usr/lib/jvm/java-17' >> ~/.bashrc
source ~/.bashrc      # Reload the file to apply changes immediately without logging out

13.11 Architecture Diagram — How PATH Resolves Commands

   You type: nginx
              │
              ▼
   Bash checks $PATH, searches each directory IN ORDER:
              │
   /usr/local/sbin ── not found here
              │
   /usr/local/bin ── not found here
              │
   /usr/sbin ── FOUND: /usr/sbin/nginx  ✓ ── Executes this one
              │
   /usr/bin ── (never checked, already found above)

This is exactly why installing two versions of a tool can cause confusing "wrong version" bugs — whichever directory appears FIRST in $PATH wins.

13.12 Best Practices


Always start scripts with #!/bin/bash and set -e (exit on any error) for production automation scripts.
Quote your variables ("$VAR" not $VAR) to avoid bugs when values contain spaces.
Add comments explaining WHY, not just WHAT, especially for non-obvious logic.
Test scripts on a non-production server first, always.
Use functions to break large scripts into readable, reusable pieces.


13.13 Common Mistakes


Forgetting chmod +x before trying to run a script with ./script.sh.
Not quoting variables, causing scripts to break on filenames/values with spaces.
Hardcoding secrets (passwords, API keys) directly in scripts instead of using environment variables or a secrets manager (AWS Secrets Manager, covered in Part 2).
Forgetting set -e, so a script continues running even after a critical command fails, potentially causing cascading damage.
Using = with spaces in variable assignment (NAME = "value" is WRONG in bash — it must be NAME="value" with NO spaces around =).


13.14 Interview Questions


What is a shebang line, and why is it necessary?
What does set -e do, and why is it important in production scripts?
What is the $PATH environment variable, and how does bash use it?
How do you make an environment variable persist across reboots/logins?
Write a bash script that checks if a service is running and restarts it if not.
What's the difference between $@ and $# in a bash script?
Why is quoting variables ("$VAR") considered a best practice?


13.15 Scenario-Based Question

Scenario: You need to write a script that takes a server name as an argument, checks if it's reachable via ping, and logs the result with a timestamp to a file. Sketch this script.

(Approach:)

bash#!/bin/bash
SERVER=$1
LOGFILE="/home/ubuntu/logs/ping_check.log"

if ping -c 2 "$SERVER" &> /dev/null; then
    echo "$(date): $SERVER is REACHABLE" >> "$LOGFILE"
else
    echo "$(date): $SERVER is UNREACHABLE" >> "$LOGFILE"
fi

13.16 Troubleshooting


Issue: bash: ./script.sh: Permission denied.
Fix: chmod +x script.sh.
Issue: Script runs but variables appear empty.
Fix: Check for spaces around = in variable assignment, or check if the variable was exported if needed by a child process.
Issue: command not found for a tool you know is installed.
Fix: Check if its install directory is in $PATH; add it via export PATH=$PATH:/new/directory if not.


13.17 Cheat Sheet

bash#!/bin/bash                  # Shebang - always first line
VAR="value"                    # Set variable (no spaces around =)
echo "$VAR"                      # Read variable
read -p "prompt: " VAR              # Get user input
$1 $2 $@ $#                          # Script arguments
if [ cond ]; then ... fi               # Conditional
for i in list; do ... done               # For loop
while [ cond ]; do ... done                # While loop
function_name() { ... }                      # Function definition
export VAR="value"                             # Set environment variable
echo $PATH                                       # View PATH
source ~/.bashrc                                   # Reload bash config
chmod +x script.sh                                   # Make executable
set -e                                                 # Exit on error

13.18 Summary

Bash scripting turns manual, repetitive Linux commands into reliable, reusable automation — the exact foundation every CI/CD pipeline is built on. Combined with environment variables for configuration, this chapter gives you everything needed to write the kind of deployment and health-check scripts real DevOps teams use daily (like the setup.sh you already built for your EC2 project).
