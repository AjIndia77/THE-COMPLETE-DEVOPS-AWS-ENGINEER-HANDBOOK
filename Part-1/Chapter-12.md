CHAPTER 12: TEXT EDITORS — vim, nano

12.1 Theory

Because most servers are headless (no GUI), you need a terminal-based text editor to view and modify configuration files directly over SSH. The two most common are nano (simple, beginner-friendly) and vim (powerful, steep learning curve, but universally available on virtually every Linux system ever built — even minimal Docker containers).

12.2 Why It Matters

Every job posting for a DevOps/Linux role assumes basic vim proficiency, because it's pre-installed everywhere and often the ONLY editor available on a minimal server or container. Interviewers frequently test at least basic vim navigation because it signals real hands-on server experience.

12.3 nano — Beginner-Friendly Editor

bashnano filename.txt

Inside nano, the bottom of the screen shows shortcuts (using ^ to mean Ctrl):


Ctrl + O — Save (Write Out)
Ctrl + X — Exit
Ctrl + K — Cut a line
Ctrl + U — Paste (Uncut)
Ctrl + W — Search (Where is)
Ctrl + \ — Search and replace


nano is straightforward — what you see is what you get, no special "modes" to worry about.

12.4 vim — The Power Editor

Vim has modes, which is the single biggest source of confusion for beginners:

ModePurposeHow to EnterNormal modeNavigate, delete, copy — default mode on openingEsc (from any other mode)Insert modeActually type/edit texti, a, o (from Normal mode)Command modeSave, quit, search-replace, run commands: (from Normal mode)Visual modeSelect text for bulk operationsv (from Normal mode)

bashvim filename.txt

Essential Vim Commands (all pressed in Normal mode unless noted):

i           Enter Insert mode BEFORE cursor
a           Enter Insert mode AFTER cursor
o           Open a new line below and enter Insert mode
Esc         Return to Normal mode from any other mode
:w          Save (write)
:q          Quit
:wq         Save and quit
:q!         Quit WITHOUT saving (force, discard changes)
:x          Save and quit (only writes if changes were made)
dd          Delete (cut) the current line
yy          Yank (copy) the current line
p           Paste after cursor
u           Undo
Ctrl + r    Redo
/searchterm Search forward for text
n           Jump to next search match
N           Jump to previous search match
:%s/old/new/g    Replace ALL occurrences of "old" with "new" in the entire file
:set number       Show line numbers
gg                Go to the very first line of the file
G                  Go to the very last line of the file
:15                Go to line 15
dG                  Delete from current line to end of file

12.5 Architecture Diagram — Vim Mode Switching

                    ┌──────────────┐
             Esc     │              │      i / a / o
        ┌───────────►│ NORMAL MODE  │◄───────────────┐
        │            │  (default)   │                 │
        │            └──────┬───────┘                 │
        │                   │ :                        │
        │                   ▼                          │
        │           ┌──────────────┐                   │
        │           │ COMMAND MODE │                    │
        │           │ :w :q :wq    │                    │
        │           └──────────────┘                    │
        │                                                │
   ┌────┴─────────┐                              ┌──────┴──────┐
   │ VISUAL MODE  │◄──────────── v ───────────────│ INSERT MODE │
   │  (select)    │                                │  (typing)   │
   └──────────────┘                                └─────────────┘

12.6 Real-World Example

You SSH into a production server to fix an Nginx config that's causing a 502 error:

bashsudo vim /etc/nginx/sites-available/default
# Press 'i' to enter insert mode, fix the proxy_pass line
# Press 'Esc', then type ':wq' to save and exit
sudo nginx -t                     # Test config syntax before reloading
sudo systemctl reload nginx        # Apply the fix

12.7 nano vs vim — When to Use Which

SituationRecommended EditorQuick one-line edit, unfamiliar servernano (if installed)Minimal Docker container / stripped-down servervim (more likely to be pre-installed)Interview / job requirementvim (expected baseline skill)Complex multi-file search-and-replacevim (:%s///g is far more powerful)

12.8 Best Practices


Always run a config syntax test (nginx -t, sshd -t) after editing critical config files, BEFORE restarting the service — this prevents locking yourself out or causing downtime.
Learn at least i, Esc, :wq, :q!, and dd in vim — this covers 90% of real-world quick edits.
Keep a backup before major edits: cp file.conf file.conf.bak before opening it.
Use :set number in vim when editing config files where line numbers matter for troubleshooting.


12.9 Common Mistakes

Opening vim, panicking, and not knowing how to exit (the classic "how do I get out of vim" meme) — remember: Esc then :q!.
Editing /etc/ssh/sshd_config incorrectly and restarting SSH, locking yourself out of the server entirely (always test with sshd -t first, and never close your existing SSH session until you've verified a NEW connection works).
Forgetting you're in Insert mode and typing commands as if in Normal mode, resulting in garbled text.


12.10 Interview Questions


What are the different modes in vim, and how do you switch between them?
How do you save and exit vim? How do you quit WITHOUT saving?
How would you search and replace all occurrences of a word in vim?
Why might vim be preferred over nano on a minimal Docker container?
What's the danger of editing sshd_config carelessly, and how do you mitigate it?


12.11 Scenario-Based Question

Scenario: You accidentally opened a critical production config file in vim, made several unwanted changes by mistake, and now need to exit without saving ANY of those changes. What exactly do you type?

(Answer: Press Esc to ensure you're in Normal mode, then type :q! and press Enter — this force-quits and discards all unsaved changes.)

12.12 Troubleshooting


Issue: Stuck in vim, can't type normally.
Fix: Press Esc to return to Normal mode, then i to safely re-enter Insert mode.
Issue: Locked out of SSH after editing sshd_config.
Fix: Use the AWS EC2 Instance Connect / Serial Console (or attach the volume to another instance) to fix the config file, since normal SSH access is now broken — always test config changes in a second SSH session before closing the first.


12.13 Cheat Sheet

nano file          # Open nano
Ctrl+O              # Save (nano)
Ctrl+X               # Exit (nano)

vim file              # Open vim
i                       # Insert mode
Esc                      # Normal mode
:wq                       # Save & quit
:q!                        # Quit without saving
dd                          # Delete line
yy                           # Copy line
p                              # Paste
/text                          # Search
:%s/old/new/g                   # Replace all

12.14 Summary

Vim and nano are your only tools for editing files on a remote, GUI-less server. Nano is friendlier for quick edits; vim is the universal standard expected in every DevOps role due to its guaranteed availability, even on the most minimal systems. Master vim's four modes and the dozen commands above, and you'll never be stuck editing a server file again.
