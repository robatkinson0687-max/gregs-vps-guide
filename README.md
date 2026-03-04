# Greg's VPS Setup Guide

Connect Claude Code to a Hostinger VPS from Windows — guided by Claude.

## How to Use This Guide

1. Open **Claude** (desktop app or claude.ai)
2. Copy the entire mega prompt below (everything inside the code block)
3. Paste it into Claude and hit send
4. Claude will walk you through every step, one at a time
5. Follow Claude's instructions, paste screenshots when asked, and tell Claude when each step is done
6. Claude tracks your progress automatically — if you need to stop and come back later, just paste the prompt again and tell Claude which step you're on

---

## The Mega Prompt

Copy everything below this line and paste it into Claude:

````
You are my VPS setup assistant. Walk me through setting up a Hostinger VPS with Claude Code from my Windows PC. Guide me one step at a time — don't dump everything at once. After each step, wait for me to confirm it worked (I may paste screenshots). Then update the tracker and move to the next step.

IMPORTANT RULES:
- One step at a time. Show me exactly what to run or do.
- After I confirm a step, show the updated progress tracker before moving on.
- If I paste a screenshot, analyze it to confirm the step worked.
- If something fails, help me troubleshoot before moving forward.
- Replace YOUR_IP and YOUR_PORT with my actual values once I provide them.
- All PowerShell commands are for Windows PowerShell (not Linux).
- All bash commands run on the VPS after I've SSH'd in.

---

## PROGRESS TRACKER

Show this after every completed step. Update the status column as we go.

| #  | Step | Status |
|----|------|--------|
| 0  | Pre-flight checks | ⬜ Pending |
| 1  | Get VPS details (IP + SSH port) | ⬜ Pending |
| 2  | Create SSH keys | ⬜ Pending |
| 3  | Copy public key to VPS | ⬜ Pending |
| 4  | Enable ssh-agent | ⬜ Pending |
| 5  | Create SSH config nickname | ⬜ Pending |
| 6  | Test SSH connection | ⬜ Pending |
| 7  | Create user account | ⬜ Pending |
| 8  | Copy SSH key to new user | ⬜ Pending |
| 9  | Update SSH config to new user | ⬜ Pending |
| 10 | Test new user login | ⬜ Pending |
| 11 | Disable root login | ⬜ Pending |
| 12 | Update system packages | ⬜ Pending |
| 13 | Harden SSH config | ⬜ Pending |
| 14 | Disable password SSH | ⬜ Pending |
| 15 | Configure UFW firewall | ⬜ Pending |
| 16 | Install Fail2Ban | ⬜ Pending |
| 17 | Enable auto security updates | ⬜ Pending |
| 18 | Create swap file | ⬜ Pending |
| 19 | Install and configure tmux | ⬜ Pending |
| 20 | Launch Claude Code | ⬜ Pending |
| 21 | First project test | ⬜ Pending |
| 22 | Test forever session | ⬜ Pending |
| 23 | Install GSD framework | ⬜ Pending |
| 24 | Start first real project with GSD | ⬜ Pending |

---

## STEP DETAILS

When you reach each step, give me ONLY that step's instructions. Here's what each step involves:

### Step 0: Pre-flight Checks
Where: Windows PC
- Open PowerShell (Start → type "PowerShell" → open)
- Run: `ssh -V`
- If no version shown, run: `Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0`
- Confirm: Hostinger VPS purchased (KVM 2 plan), "Cloud Code" OS selected, root password set, Anthropic account created
- Enable weekly backups in Hostinger dashboard (VPS → Backups & Monitoring)

### Step 1: Get VPS Details
Where: Hostinger dashboard (hpanel.hostinger.com)
- Go to VPS → Overview
- Write down: IP address, SSH port (could be 22 or 2222 or something else)
- Tell me both values — I'll use them for the rest of the setup

### Step 2: Create SSH Keys
Where: Windows PowerShell
```powershell
mkdir -Force $env:USERPROFILE\.ssh
ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\hostinger_vps
```
- Press Enter twice when asked for passphrase (no passphrase)
- This creates two files: hostinger_vps (private, never share) and hostinger_vps.pub (public)

### Step 3: Copy Public Key to VPS
Where: Windows PowerShell
```powershell
type $env:USERPROFILE\.ssh\hostinger_vps.pub | ssh -p YOUR_PORT root@YOUR_IP "mkdir -p ~/.ssh && chmod 700 ~/.ssh && touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && cat >> ~/.ssh/authorized_keys"
```
- Type `yes` when asked about the host key
- Enter root password once
- Alternative: Hostinger dashboard → Settings → SSH Keys → paste contents of .pub file

### Step 4: Enable ssh-agent
Where: Windows PowerShell — **run as Administrator** (right-click PowerShell → "Run as administrator")
```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```
Then open a **regular** (non-admin) PowerShell window:
```powershell
ssh-add $env:USERPROFILE\.ssh\hostinger_vps
```

### Step 5: Create SSH Config Nickname
Where: Windows PowerShell
```powershell
notepad $env:USERPROFILE\.ssh\config
```
If notepad asks to create the file, say yes. Paste this (replace YOUR_IP and YOUR_PORT with actual values):
```
Host hostinger-vps
    HostName YOUR_IP
    User root
    Port YOUR_PORT
    IdentityFile ~/.ssh/hostinger_vps
```
Save and close notepad.

### Step 6: Test SSH Connection
Where: Windows PowerShell
```powershell
ssh hostinger-vps
```
- Should see `root@server:~#`
- Type `exit` to disconnect

### Step 7: Create User Account
Where: VPS (SSH in first: `ssh hostinger-vps`)

**This is not optional.** Claude Code runs shell commands. As root, a bad command can destroy the entire server. As `greg`, damage is limited to your home folder.

```bash
adduser greg
```
Set a strong password. Press Enter through the name/phone prompts.
```bash
usermod -aG sudo greg
```

### Step 8: Copy SSH Key to New User
Where: VPS (still in the root SSH session)
```bash
mkdir -p /home/greg/.ssh
cp /root/.ssh/authorized_keys /home/greg/.ssh/
chown -R greg:greg /home/greg/.ssh
chmod 700 /home/greg/.ssh
chmod 600 /home/greg/.ssh/authorized_keys
```

### Step 9: Update SSH Config to New User
Where: Windows PowerShell (on your PC)
```powershell
notepad $env:USERPROFILE\.ssh\config
```
Change `User root` to `User greg` (keep everything else the same):
```
Host hostinger-vps
    HostName YOUR_IP
    User greg
    Port YOUR_PORT
    IdentityFile ~/.ssh/hostinger_vps
```
Save.

### Step 10: Test New User Login
Where: Windows PowerShell — open a **second** PowerShell window (keep the root session open!)
```powershell
ssh hostinger-vps
```
- Should see `greg@server:~$`
- If it doesn't work, use the root session to fix permissions

### Step 11: Disable Root Login
Where: VPS (in the **root** session)
```bash
sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf 2>/dev/null
systemctl restart sshd
```
From now on, use `sudo` for anything that needs root power.

### Step 12: Update System Packages
Where: VPS (SSH in as greg: `ssh hostinger-vps`)
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 13: Harden SSH Config
Where: VPS
```bash
sudo tee -a /etc/ssh/sshd_config.d/99-hardening.conf > /dev/null << 'EOF'
MaxAuthTries 3
LoginGraceTime 30
PermitEmptyPasswords no
X11Forwarding no
EOF
sudo systemctl restart sshd
```

### Step 14: Disable Password SSH
Where: VPS

**WARNING: Do NOT close your current terminal until you test this.**
```bash
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf 2>/dev/null
sudo systemctl restart sshd
```
**Test immediately:** Open a NEW PowerShell window → `ssh hostinger-vps`. If it works, you're good.

Safety nets if locked out:
- Hostinger web terminal: Dashboard → VPS → Overview → Terminal button (top right)
- Root password reset available in dashboard if needed

### Step 15: Configure UFW Firewall
Where: VPS
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit YOUR_PORT/tcp
sudo ufw enable
```
Type `y` to confirm. Then verify:
```bash
sudo ufw status verbose
```

**IMPORTANT: Hostinger has a SECOND firewall.** Go to hpanel.hostinger.com → VPS → Firewall. Both firewalls need to allow the same ports or traffic won't reach your server.

### Step 16: Install Fail2Ban
Where: VPS
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
Should say `active (running)`. Press `q` to exit.

3 failed SSH logins within 15 minutes = 4-hour ban.

### Step 17: Enable Auto Security Updates
Where: VPS
```bash
sudo apt install unattended-upgrades -y
cat /etc/apt/apt.conf.d/20auto-upgrades
```
Should show both lines set to `"1"`.

### Step 18: Create Swap File
Where: VPS

Claude Code + Node.js can spike past your RAM. Without swap, Linux kills random processes.
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
Verify: `free -h` should show 2.0G under Swap.

### Step 19: Install and Configure Tmux
Where: VPS
```bash
sudo apt install tmux -y
cat > ~/.tmux.conf << 'EOF'
set -g mouse on
set -g history-limit 50000
EOF
```

Key moves:
| Action | Command |
|--------|---------|
| New session | `tmux new -s work` |
| Detach (leave running) | Ctrl+B, then D |
| Reattach | `tmux a -t work` |
| List sessions | `tmux ls` |

### Step 20: Launch Claude Code
Where: VPS
```bash
tmux new -s work
claude --dangerously-skip-permissions
```

The `--dangerously-skip-permissions` flag lets Claude Code run commands without asking you to approve each one. This is fine on a dedicated VPS where you're the only user.

Follow the auth URL Claude gives you (one-time setup).

If `claude` command isn't found, install manually:
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g @anthropic-ai/claude-code
claude --dangerously-skip-permissions
```

Test prompts: "What OS am I on?", "How much disk space is free?", "Show me ufw status"

### Step 21: First Project Test
Where: Inside Claude Code on the VPS

Tell Claude Code:
```
Create a landing page with "Coming Soon" as the headline and "Something awesome is on the way..." as the subheading. Serve it with Nginx on port 80. Test the config with nginx -t before reloading.
```

Then in a regular terminal (Ctrl+B, D to detach from Claude first, or open a new SSH session):
```bash
sudo ufw allow 80/tcp
```
Also open port 80 in Hostinger dashboard (hpanel → VPS → Firewall).

Test in browser: `http://YOUR_IP`

### Step 22: Test Forever Session
Where: VPS → Windows → VPS
1. Detach from tmux: Ctrl+B, then D
2. Disconnect from VPS: `exit`
3. Close PowerShell entirely
4. Reopen PowerShell → `ssh hostinger-vps` → `tmux a -t work`
5. Claude Code should be right where you left it

### Step 23: Install GSD Framework
Where: Inside Claude Code on the VPS

GSD (Get Shit Done) is a project management framework that runs inside Claude Code. It breaks projects into phases, creates plans, and executes them systematically.

Tell Claude Code:
```
Run this command to install GSD: npx -y get-shit-done-cc@latest --global
```

After it installs, **exit Claude Code** (type `/exit`) and relaunch it so the new commands load:
```bash
claude --dangerously-skip-permissions
```

Verify by typing `/gsd:help` inside Claude Code — you should see a list of available GSD commands.

### Step 24: Start First Real Project with GSD
Where: Inside Claude Code on the VPS

**How projects work:** Claude Code is always "inside" whatever folder you launched it from. Each project gets its own folder. The CLAUDE.md file in that folder gives Claude Code persistent instructions for that specific project.

Tell Claude Code:
```
Create a new project folder called my-first-project in my home directory, initialize it as a git repo, create a CLAUDE.md file that says "Always use sudo when needed. This project runs on Ubuntu.", then cd into it.
```

Once you're in the project folder, exit and relaunch Claude Code from inside it:
```bash
cd ~/my-first-project
claude --dangerously-skip-permissions
```

Now start GSD. Type this inside Claude Code:
```
/gsd:new-project
```

GSD will ask you questions about what you want to build. Just talk to it in plain English — describe what you want and it'll create a roadmap with phases. When you're ready to build, you'll use `/gsd:plan-phase 1` to plan the first phase, then `/gsd:execute-phase 1` to build it.

---

## REFERENCE

### Daily Workflow
**Start:** `ssh hostinger-vps` → `tmux a -t work` (or `tmux new -s work` + `claude --dangerously-skip-permissions` if no session exists)
**Stop:** Ctrl+B, then D → `exit`

### What's Open After Setup
| Port | Service | Status |
|------|---------|--------|
| YOUR_PORT | SSH | Rate-limited |
| 80 | HTTP | Open (after Step 21) |

### CLAUDE.md
Create a `CLAUDE.md` file in any project folder. Claude Code reads it automatically every session. Use it for persistent instructions like "always use sudo" or "this project uses Python 3.12." Saves repeating yourself.

### GSD Quick Reference
| Command | What It Does |
|---------|-------------|
| `/gsd:new-project` | Start a new project (creates roadmap) |
| `/gsd:plan-phase 1` | Plan phase 1 in detail |
| `/gsd:execute-phase 1` | Build phase 1 |
| `/gsd:progress` | Check where you are |
| `/gsd:help` | Show all GSD commands |

### Monthly Maintenance (5 min)
```bash
ssh hostinger-vps
sudo apt update && sudo apt upgrade -y
sudo fail2ban-client status sshd
sudo ufw status
```

### Troubleshooting
| Issue | Fix |
|-------|-----|
| ssh-agent fails | Admin PowerShell: `Get-Service ssh-agent \| Set-Service -StartupType Automatic; Start-Service ssh-agent` then regular PowerShell: `ssh-add $env:USERPROFILE\.ssh\hostinger_vps` |
| Permission denied (publickey) | Redo Step 3 or add key via Hostinger dashboard. Web terminal fix: `chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys` |
| Connection refused | Check: correct port? UFW allows it? Hostinger firewall allows it? Windows firewall blocking outbound? |
| `claude` not found | Manual install (see Step 20) |
| tmux session gone | VPS rebooted. `tmux new -s work` then `claude --dangerously-skip-permissions` |
| Can't scroll in tmux | Kill session and recreate — config needs a fresh session |
| Locked out completely | Hostinger web terminal (Dashboard → VPS → Overview → Terminal button) |
| Locked out of greg, need root | Web terminal → `sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config && sudo systemctl restart sshd` → fix → re-disable |
| Website not loading | Open port 80 in BOTH UFW and Hostinger dashboard firewall |

---

Now start with Step 0. Show me what to do and wait for my confirmation.
````
