# Greg's VPS Setup Guide

Set up your own cloud server with Claude Code — no experience needed.

## How to Use This Guide

1. Open **Claude** (desktop app or claude.ai)
2. Copy the entire mega prompt below (everything inside the code block)
3. Paste it into Claude and hit send
4. Claude walks you through every step, one at a time — just follow along
5. You can paste screenshots at any point and Claude will check your work
6. Need to stop? Come back later, paste the prompt again, and tell Claude which step you left off on

---

## The Mega Prompt

Copy everything below this line and paste it into Claude:

````
You are walking a complete beginner through setting up a cloud server (VPS) with Claude Code. I'm on Windows. I have zero Linux or server experience. Guide me one step at a time.

RULES FOR YOU:
- One step at a time. Don't show me the next step until I confirm the current one worked.
- Explain what each command does in plain English before I run it. No jargon without explanation.
- After I confirm a step, show the updated progress tracker, then give me the next step.
- If I paste a screenshot, look at it and tell me if things look right.
- If something fails or looks wrong, help me fix it before moving on. Don't skip ahead.
- If something fails, ask me to run the failed command again and show you the exact error output (or paste a screenshot). Diagnose what went wrong before suggesting a fix — don't guess.
- Once I give you my IP address and SSH port in Step 1, use those real values in every command from that point on. No more placeholders.
- If any step says something "already exists" or gives unexpected output, ask me to show you what it says — the Hostinger Cloud Code image may have pre-configured some things.
- Commands labeled "Windows PowerShell" run on my PC. Commands labeled "VPS" run on the server after I've connected to it.
- Keep it encouraging. This is my first time doing anything like this.

---

## PROGRESS TRACKER

Show this updated table after every completed step.

| #  | Step | Status |
|----|------|--------|
| 0  | Pre-flight checks (stuff to buy/sign up for) | ⬜ Pending |
| 1  | Get your server's IP and port | ⬜ Pending |
| 2  | Create your SSH keys (like a digital key pair) | ⬜ Pending |
| 3  | Send your key to the server | ⬜ Pending |
| 4  | Set up key manager on Windows | ⬜ Pending |
| 5  | Create a shortcut name for your server | ⬜ Pending |
| 6  | Test the connection | ⬜ Pending |
| 7  | Create your own user account on the server | ⬜ Pending |
| 8  | Give your new account the same key access | ⬜ Pending |
| 9  | Update Windows to use your new account | ⬜ Pending |
| 10 | Test logging in as yourself | ⬜ Pending |
| 11 | Lock out the root account from remote login | ⬜ Pending |
| 12 | Update all server software | ⬜ Pending |
| 13 | Tighten up SSH security | ⬜ Pending |
| 14 | Turn off password logins (keys only) | ⬜ Pending |
| 15 | Turn on the firewall | ⬜ Pending |
| 16 | Install brute-force protection | ⬜ Pending |
| 17 | Turn on automatic security updates | ⬜ Pending |
| 18 | Add emergency memory (swap file) | ⬜ Pending |
| 19 | Install tmux (keeps sessions alive) | ⬜ Pending |
| 20 | Launch Claude Code for the first time | ⬜ Pending |
| 21 | Build a test webpage to prove it works | ⬜ Pending |
| 22 | Test that your session survives disconnecting | ⬜ Pending |
| 23 | Set up global CLAUDE.md (your permanent instructions) | ⬜ Pending |
| 24 | Install the GSD project framework | ⬜ Pending |
| 25 | Start your first real project | ⬜ Pending |

---

## WHAT IS ALL THIS STUFF?

Before we start, here's the big picture in plain English:

**VPS (Virtual Private Server):** A computer in the cloud that's always on. You're renting one from Hostinger. Claude Code lives on this computer, not on your Windows PC.

**SSH:** A secure way to control a remote computer by typing commands. Think of it like a phone call to your server — encrypted so nobody can eavesdrop.

**SSH Keys:** Instead of a password, you use a matched pair of digital keys. One stays on your PC (private key — never share this). The other goes on the server (public key). They work together like a lock and key.

**Tmux:** A tool that keeps your Claude Code session running even if you close your laptop or lose internet. When you reconnect, everything is right where you left it.

**CLAUDE.md:** A text file you create that gives Claude Code standing instructions. Think of it like a note pinned to Claude's desk that it reads every time it starts up. More on this later.

---

## STEP DETAILS

### Step 0: Pre-flight Checks
Where: Your Windows PC + web browser

Things to do before we start:
- Open PowerShell on your PC: Start menu → type "PowerShell" → click to open
- Run this to make sure SSH is installed: `ssh -V`
- If you see a version number, you're good. If you get an error, run: `Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0`
- Buy a Hostinger VPS: go to hostinger.com → VPS Hosting → pick the KVM 2 plan
- When setting up the VPS, choose **"Cloud Code"** as the operating system (this pre-installs Claude Code)
- Set a strong root password (16+ characters) during creation — you'll need it briefly
- Create an Anthropic account at anthropic.com (you need credits or a plan to use Claude Code). Check your usage and billing at console.anthropic.com — Claude Code stops working when credits run out, so keep an eye on it
- In Hostinger dashboard: go to VPS → Backups & Monitoring → turn on weekly backups

Tell me when you've done all of this and I'll move to Step 1.

### Step 1: Get Your Server's IP and Port
Where: Hostinger dashboard (hpanel.hostinger.com)

- Go to VPS → Overview
- Find two things: your **IP address** (looks like 123.45.67.89) and your **SSH port** (usually 22 or 2222)
- Tell me both values — I'll plug them into every command from here on

### Step 2: Create SSH Keys
Where: Windows PowerShell

This creates your digital key pair. The private key stays on your PC. The public key will go to the server.
```powershell
mkdir -Force $env:USERPROFILE\.ssh
ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\hostinger_vps
```
- It will ask for a passphrase — just press Enter twice (no passphrase, keeps things simple)
- You'll see two new files created: `hostinger_vps` (private — guard this) and `hostinger_vps.pub` (public — safe to share)

### Step 3: Send Your Key to the Server
Where: Windows PowerShell + Hostinger Dashboard

This copies your public key to the server so it recognizes you. We'll use the **easiest method first** (the dashboard), with a command-line backup if needed.

**Method A — Dashboard (recommended):**
1. In PowerShell, run this to display your public key:
   ```powershell
   type $env:USERPROFILE\.ssh\hostinger_vps.pub
   ```
2. Copy the entire output (starts with `ssh-ed25519`)
3. Go to Hostinger dashboard → VPS → Settings → SSH Keys
4. Paste your key there and save

**Method B — Command line (if the dashboard option isn't available):**
Replace YOUR_PORT and YOUR_IP with your actual values from Step 1:
```powershell
type $env:USERPROFILE\.ssh\hostinger_vps.pub | ssh -p YOUR_PORT root@YOUR_IP "mkdir -p ~/.ssh && chmod 700 ~/.ssh && touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && cat >> ~/.ssh/authorized_keys"
```
- It will ask "Are you sure you want to continue connecting?" — type `yes`
- Enter your root password one time

### Step 4: Set Up Key Manager on Windows
Where: Windows PowerShell

This tells Windows to remember your key so you don't have to point to the file every time.

First, open PowerShell **as Administrator** (right-click PowerShell → "Run as administrator"):
```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Then open a **regular** (non-admin) PowerShell window and run:
```powershell
ssh-add $env:USERPROFILE\.ssh\hostinger_vps
```

**If ssh-agent gives you trouble, don't panic.** The SSH config file we create in Step 5 points directly to your key file (`IdentityFile`), so connections will still work without ssh-agent. It's a convenience, not a requirement.

### Step 5: Create a Shortcut Name for Your Server
Where: Windows PowerShell

Instead of typing the IP address every time, we'll create a nickname. Run:
```powershell
notepad $env:USERPROFILE\.ssh\config
```
If notepad asks to create the file, say yes. Paste this (replace YOUR_IP and YOUR_PORT):
```
Host hostinger-vps
    HostName YOUR_IP
    User root
    Port YOUR_PORT
    IdentityFile ~/.ssh/hostinger_vps
```
Save and close notepad. Now you can type `ssh hostinger-vps` instead of remembering the IP.

### Step 6: Test the Connection
Where: Windows PowerShell
```powershell
ssh hostinger-vps
```
- You should see something like `root@server:~#` — that means you're inside your server
- Type `exit` to disconnect and come back to Windows
- If it asks for a password, something went wrong in Steps 3-4 — tell me what happened

### Step 7: Create Your Own User Account
Where: VPS (SSH in first: `ssh hostinger-vps`)

Right now you're logging in as `root`, which is like being the administrator of the server — full power to do anything, including accidentally breaking everything. We're going to create a normal user account for you. Claude Code will use this account, which limits the blast radius if something goes wrong.

```bash
adduser greg
```
Set a strong password when asked. It will ask for your full name, phone, etc. — just press Enter through all of those.

Then give your account the ability to run admin commands when needed:
```bash
usermod -aG sudo greg
```

### Step 8: Give Your New Account the Same Key Access
Where: VPS (still connected as root)

Copy your SSH key access to the new account so you can log in as greg without a password too:
```bash
mkdir -p /home/greg/.ssh
cp /root/.ssh/authorized_keys /home/greg/.ssh/
chown -R greg:greg /home/greg/.ssh
chmod 700 /home/greg/.ssh
chmod 600 /home/greg/.ssh/authorized_keys
```

### Step 9: Update Windows to Use Your New Account
Where: Windows PowerShell (on your PC — not the server)
```powershell
notepad $env:USERPROFILE\.ssh\config
```
Change `User root` to `User greg`. Keep everything else the same:
```
Host hostinger-vps
    HostName YOUR_IP
    User greg
    Port YOUR_PORT
    IdentityFile ~/.ssh/hostinger_vps
```
Save.

### Step 10: Test Logging In As Yourself
Where: Windows PowerShell

**Important: keep your current root session open.** Open a **second** PowerShell window and run:
```powershell
ssh hostinger-vps
```
- You should see `greg@server:~$` (notice it says greg now, not root)
- If this works, great — move on. If not, use the root session to fix it

### Step 11: Lock Out the Root Account from Remote Login
Where: VPS (in your **root** session — the first PowerShell window)

Now that you can log in as greg, we'll prevent anyone from logging in as root remotely. This is a big security win.
```bash
sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf 2>/dev/null
systemctl restart sshd
```
From now on, if you need admin powers, use `sudo` before a command (e.g., `sudo apt install something`). When Claude Code needs admin power, just tell it "use sudo when needed."

### Step 12: Update All Server Software
Where: VPS (SSH in as greg: `ssh hostinger-vps`)
```bash
sudo apt update && sudo apt upgrade -y
```
This downloads and installs the latest versions of everything. It might take a minute. When it asks for your password, use the greg password you set in Step 7.

### Step 13: Tighten Up SSH Security
Where: VPS

This limits login attempts and turns off features you don't need:
```bash
sudo tee -a /etc/ssh/sshd_config.d/99-hardening.conf > /dev/null << 'EOF'
MaxAuthTries 3
LoginGraceTime 30
PermitEmptyPasswords no
X11Forwarding no
EOF
sudo systemctl restart sshd
```

### Step 14: Turn Off Password Logins (Keys Only)
Where: VPS

**WARNING: Do NOT close your current PowerShell window until you test this.** If something goes wrong, you need this session to fix it.

```bash
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf 2>/dev/null
sudo systemctl restart sshd
```

**Test immediately:** Open a brand new PowerShell window → `ssh hostinger-vps`. If you get in, you're good.

**Verify no other config file is overriding this:** Run this to check:
```bash
sudo grep -r PasswordAuthentication /etc/ssh/
```
Every line should say `no`. If any file still says `yes`, tell me which file and I'll help you fix it. (Hostinger's Cloud Code image sometimes has drop-in config files that override the main setting.)

If you get locked out:
- **Hostinger web terminal:** Dashboard → VPS → Overview → Terminal button (top right corner). This always works regardless of SSH settings.
- Root password reset is also available in the dashboard.

### Step 15: Turn On the Firewall
Where: VPS

This blocks all incoming traffic except SSH (so you can still connect):
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit YOUR_PORT/tcp
sudo ufw enable
```
Type `y` to confirm. Then check it's working:
```bash
sudo ufw status verbose
```

**IMPORTANT: Hostinger has a SECOND firewall** in their dashboard. Go to hpanel.hostinger.com → VPS → Firewall. If a port is open in one firewall but not the other, traffic gets blocked. You need both to agree.

### Step 16: Install Brute-Force Protection
Where: VPS

Fail2Ban watches for repeated failed login attempts and temporarily bans the attacker's IP:
```bash
sudo apt install fail2ban -y
sudo tee /etc/fail2ban/jail.local > /dev/null << 'EOF'
[DEFAULT]
bantime  = 3600
findtime = 600
maxretry = 5
backend  = systemd
banaction = ufw

[sshd]
enabled  = true
port     = YOUR_PORT
maxretry = 3
findtime = 900
bantime  = 14400
EOF
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
```
Should say `active (running)`. Press `q` to exit that screen.

In plain English: 3 failed login attempts within 15 minutes = banned for 4 hours.

### Step 17: Turn On Automatic Security Updates
Where: VPS
```bash
sudo apt install unattended-upgrades -y
cat /etc/apt/apt.conf.d/20auto-upgrades
```
Should show two lines both set to `"1"`. This means critical security patches install themselves — you don't have to remember to do it.

### Step 18: Add Emergency Memory (Swap File)
Where: VPS

Sometimes Claude Code and Node.js use more memory than your server has. Without a swap file, Linux just kills whatever's using too much. The swap file acts like overflow parking — slower, but keeps things alive.
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
Verify: `free -h` — you should see 2.0G under the Swap row.

### Step 19: Install Tmux (Keeps Sessions Alive)
Where: VPS

Tmux lets you start a session, disconnect, close your laptop, and come back later to find everything exactly where you left it. Essential for Claude Code.
```bash
sudo apt install tmux -y
cat > ~/.tmux.conf << 'EOF'
set -g mouse on
set -g history-limit 50000
EOF
```

Here are the only tmux commands you need to know:
| What you want to do | What to type |
|---------------------|-------------|
| Start a new session | `tmux new -s work` |
| Leave the session running and go back to normal terminal | Press Ctrl+B, then press D |
| Come back to your session later | `tmux a -t work` |
| See what sessions exist | `tmux ls` |

### Step 20: Launch Claude Code for the First Time
Where: VPS

This is the moment — start a tmux session and launch Claude Code:
```bash
tmux new -s work
claude --dangerously-skip-permissions
```

What's that `--dangerously-skip-permissions` flag? Normally Claude Code asks your permission before running every single command. On your own dedicated server where you're the only user, this is unnecessary friction. This flag tells Claude Code to just go ahead and run things. You're in a safe environment.

**One thing to know:** With this flag, Claude Code won't ask before running commands. So read what it says it's about to do before you hit enter. If it says something like "I'm going to delete..." or "I'm going to modify your SSH config..." — make sure that's actually what you want. We'll add safety rails in the CLAUDE.md file (Step 23) to prevent the worst-case scenarios.

Claude will give you a URL to open in your browser for authentication. Do that once — it remembers you after.

If the `claude` command isn't found (the Cloud Code image didn't install it), do this:
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g @anthropic-ai/claude-code
claude --dangerously-skip-permissions
```

Try some test prompts to make sure it's working:
- "What OS am I on?"
- "How much disk space is free?"
- "Show me ufw status"

### Step 21: Build a Test Webpage
Where: Inside Claude Code on the VPS

Tell Claude Code:
```
Create a landing page with "Coming Soon" as the headline and "Something awesome is on the way..." as the subheading. Serve it with Nginx on port 80. Test the config with nginx -t before reloading.
```

Then open the firewall for web traffic. You can either tell Claude Code to do this, or detach from tmux (Ctrl+B, D) and run it yourself:
```bash
sudo ufw allow 80/tcp
```
Also open port 80 in the Hostinger dashboard firewall (hpanel → VPS → Firewall).

Open a browser on your PC and go to: `http://YOUR_IP`

You should see your "Coming Soon" page. You just built and deployed a website from plain English. That's what this whole setup is for.

### Step 22: Test That Your Session Survives Disconnecting
Where: VPS → Windows → VPS

This proves tmux is working — your Claude Code session stays alive even when you disconnect:
1. Press Ctrl+B, then D (detaches from tmux — Claude Code keeps running in the background)
2. Type `exit` (disconnects from the server)
3. Close PowerShell entirely
4. Open a new PowerShell → `ssh hostinger-vps` → `tmux a -t work`
5. Claude Code should be right where you left it, mid-conversation

### Step 23: Set Up Global CLAUDE.md
Where: VPS

**This is important to understand.** CLAUDE.md is a text file that gives Claude Code permanent standing instructions. It reads this file every time it starts up.

There are two levels:
1. **Global CLAUDE.md** (`~/CLAUDE.md`) — Instructions that apply to EVERY project, no matter what folder you're in. Set it up once.
2. **Project CLAUDE.md** (`~/some-project/CLAUDE.md`) — Instructions that only apply when you're working inside that specific project folder. You'll create these per-project.

They stack: when you're in a project folder, Claude Code reads BOTH the global one AND the project one.

Let's create your global one first. Tell Claude Code:
```
Create a file at ~/CLAUDE.md with this content:

# Global Instructions

## Safety Rules
- NEVER delete system files, modify SSH config, or change firewall rules without explaining exactly what you're about to do and waiting for my confirmation
- NEVER run rm -rf on system directories
- Before any destructive operation (deleting files, overwriting configs, killing processes), tell me what you're about to do and why

## How I Work
- Always use sudo when a command needs root privileges
- This server runs Ubuntu on a Hostinger VPS
- When serving web projects, always use port 3000 or port 80
- Explain what you're doing before you do it — I'm still learning
```

You can always edit this file later to add more instructions as you figure out your preferences.

### Step 24: Install the GSD Project Framework
Where: Inside Claude Code on the VPS

GSD (Get Shit Done) is a framework that runs inside Claude Code and helps you manage projects. Instead of just asking Claude Code random questions, GSD gives your project structure: it breaks work into phases, creates plans, builds them, and verifies the results.

Tell Claude Code:
```
Run this command to install GSD: npx -y get-shit-done-cc@latest --global
```

After it installs, exit Claude Code (type `/exit`) and relaunch so the new commands load:
```bash
claude --dangerously-skip-permissions
```

Type `/gsd:help` inside Claude Code to verify it's installed — you should see a list of commands.

### Step 25: Start Your First Real Project
Where: VPS terminal + Claude Code

Every project you build gets its own folder. When you launch Claude Code from inside a project folder, it knows that's the project you're working on. It reads the global `~/CLAUDE.md` plus any project-specific `CLAUDE.md` in that folder.

Here's the workflow. First, create a project folder and go into it:
```bash
mkdir ~/my-first-project
cd ~/my-first-project
git init
```

Launch Claude Code inside that folder:
```bash
claude --dangerously-skip-permissions
```

Now tell Claude Code to set up a project-level CLAUDE.md:
```
Create a CLAUDE.md file in this folder that says "This is a [describe your project]. Always use sudo when needed."
```

Then start GSD by typing:
```
/gsd:new-project
```

GSD will ask you questions about what you want to build. Just describe it in plain English — like explaining it to a smart friend. It creates a roadmap broken into phases.

When you're ready to start building:
- `/gsd:plan-phase 1` — makes a detailed plan for the first phase
- `/gsd:execute-phase 1` — actually builds it
- `/gsd:progress` — shows where you are

That's it. You now have a cloud server, Claude Code, and a project management system. Just describe what you want to build and let Claude do the heavy lifting.

---

## REFERENCE (don't need this during setup — it's for later)

### Daily Workflow
```
ssh hostinger-vps
tmux a -t work
```
If no tmux session exists: `tmux new -s work` then `claude --dangerously-skip-permissions`

When you're done: Ctrl+B, then D, then `exit`

### Restarting Claude Code
If Claude Code freezes, crashes, or you need a fresh start:
- Type `/exit` or press Ctrl+C to quit
- Then run `claude --dangerously-skip-permissions` to start it again
- Your tmux session stays alive — only Claude Code restarts

### Starting a New Project
```bash
mkdir ~/project-name
cd ~/project-name
git init
claude --dangerously-skip-permissions
```
Then inside Claude Code: create a CLAUDE.md and run `/gsd:new-project`

### CLAUDE.md Cheat Sheet
| File | Scope | Example |
|------|-------|---------|
| `~/CLAUDE.md` | Everything, always | "Always use sudo when needed" |
| `~/my-project/CLAUDE.md` | Only that project | "This project uses Python 3.12" |
Both are read when you're inside a project folder.

### GSD Commands
| Command | What It Does |
|---------|-------------|
| `/gsd:new-project` | Describe what you want to build, get a roadmap |
| `/gsd:plan-phase 1` | Plan phase 1 in detail |
| `/gsd:execute-phase 1` | Build phase 1 |
| `/gsd:progress` | See where you are |
| `/gsd:help` | Show all GSD commands |

### What Ports Are Open
| Port | Service | Status |
|------|---------|--------|
| YOUR_PORT | SSH | Rate-limited |
| 80 | HTTP | Open (after Step 21) |
Everything else is blocked. Open more with `sudo ufw allow PORT/tcp` + Hostinger dashboard.

### Monthly Maintenance (5 min)
```bash
ssh hostinger-vps
sudo apt update && sudo apt upgrade -y
sudo fail2ban-client status sshd
sudo ufw status
```

### Troubleshooting
| Problem | Fix |
|---------|-----|
| ssh-agent fails on Windows | Admin PowerShell: `Get-Service ssh-agent \| Set-Service -StartupType Automatic; Start-Service ssh-agent` then regular PowerShell: `ssh-add $env:USERPROFILE\.ssh\hostinger_vps` |
| "Permission denied (publickey)" | Redo Step 3 or add key via Hostinger dashboard (Settings → SSH Keys). If on the web terminal: `chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys` |
| "Connection refused" | Wrong port? UFW blocking it? Hostinger firewall blocking it? Windows firewall blocking outbound? |
| `claude` command not found | See manual install in Step 20 |
| Tmux session disappeared | Server probably rebooted. `tmux new -s work` then `claude --dangerously-skip-permissions` |
| Can't scroll in tmux | Kill the session (`tmux kill-session -t work`) and create a new one — the config only loads on fresh sessions |
| Completely locked out | Hostinger web terminal: Dashboard → VPS → Overview → Terminal button (top right). Always works. Root password reset available in dashboard. |
| Need root access back temporarily | Web terminal → `sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config && sudo systemctl restart sshd` → fix your issue → re-disable root login |
| Website not loading in browser | Did you open port 80 in BOTH UFW (`sudo ufw allow 80/tcp`) AND the Hostinger dashboard firewall? |
| Claude Code stopped responding or won't start | Check your Anthropic credits at console.anthropic.com. Also try: `/exit` or Ctrl+C, then `claude --dangerously-skip-permissions` |

---

Now start with Step 0. Show me what to do and wait for my confirmation before moving on.
````
