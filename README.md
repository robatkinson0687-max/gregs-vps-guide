# Greg's VPS Setup Guide (Beginner-Friendly, Actually Secure)

Connect Claude Code to a Hostinger VPS from Windows
**Follow every step in order. Don't skip anything.**

***

## How This Whole Thing Works

You're going to rent a computer in the cloud (a VPS) from Hostinger. Claude Code will be installed on that **cloud computer**, not on your Windows PC. You'll use Windows PowerShell to remotely connect over secure SSH. Once connected, launch Claude Code on the VPS and chat in plain English to build things.

**Key trick:** Tmux keeps your Claude Code session alive even if you close your laptop or lose internet. Reconnect and pick up exactly where you left off.

**Safety nets:**
- **Hostinger dashboard terminal:** If SSH breaks, VPS → Overview → Terminal button (top right). Always works.
- **Snapshots/backups:** Enable weekly backups and create snapshots before big changes.

***

## Before You Start

1. **Windows PowerShell:** Start → type "PowerShell" → open.
2. **Verify OpenSSH is installed:** Run this in PowerShell:
   ```powershell
   ssh -V
   ```
   If you see a version number, you're good. If not (or you get an error), install it:
   ```powershell
   Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
   ```
   (Windows 11 and recent Windows 10 have it pre-installed. Older Windows 10 may not.)
3. **Hostinger VPS:** Buy KVM 2 plan at hostinger.com.
4. **OS:** Select **"Cloud Code"** during setup (pre-installs Claude Code on Ubuntu).
5. **Root password:** Set a strong one (16+ chars) during creation.
6. **Anthropic account:** Sign up at anthropic.com with credits/plan.
7. **Backups:** VPS dashboard → Backups & Monitoring → Enable weekly + create initial snapshot.

***

## Phase 1: SSH Key Setup (Passwordless Login)

### Step 1: Get VPS Details
Dashboard → VPS → Overview:
- IP address (e.g., 123.45.67.89)
- Username: `root`
- SSH Port (e.g., 22 or 2222 — **write this down**, you'll need it in several steps)

### Step 2: Create Keys
PowerShell:
```powershell
mkdir -Force $env:USERPROFILE\.ssh
ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\hostinger_vps
```
Press Enter twice (no passphrase for simplicity). The `mkdir -Force` creates the .ssh folder if it doesn't exist (safe to run if it already does).

Creates two files: `hostinger_vps` (private — never share), `hostinger_vps.pub` (public).

### Step 3: Copy Public Key to VPS
PowerShell (one command). Use the SSH port from Step 1 — if it's 22, you can omit `-p`. If it's something else (like 2222), include `-p YOUR_PORT`:

```powershell
type $env:USERPROFILE\.ssh\hostinger_vps.pub | ssh -p YOUR_PORT root@YOUR_IP "mkdir -p ~/.ssh && chmod 700 ~/.ssh && touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && cat >> ~/.ssh/authorized_keys"
```

- Type `yes` to accept the host key.
- Enter root password once.

**Dashboard alternative:** Settings → SSH Keys → Paste the contents of your `.pub` file.

### Step 4: Enable ssh-agent
PowerShell (**run as Administrator** — right-click PowerShell → "Run as administrator"):
```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Then in a **regular** PowerShell window:
```powershell
ssh-add $env:USERPROFILE\.ssh\hostinger_vps
```

### Step 5: SSH Nickname
PowerShell:
```powershell
notepad $env:USERPROFILE\.ssh\config
```

If notepad asks to create the file, say yes. Paste (replace YOUR_IP and YOUR_PORT):
```
Host hostinger-vps
    HostName YOUR_IP
    User root
    Port YOUR_PORT
    IdentityFile ~/.ssh/hostinger_vps
```
Save and close.

### Step 6: Test
```powershell
ssh hostinger-vps
```
You should see `root@server:~#` → type `exit` to disconnect.

***

## Phase 2: Create Your User Account

**This is not optional.** Claude Code runs shell commands. As root, a bad command can destroy the entire server. As `greg`, damage is limited to your home folder. System files stay protected.

### Step 7: Create User
SSH in: `ssh hostinger-vps`

```bash
adduser greg
```
Set a strong password. Press Enter through the name/phone prompts.

```bash
usermod -aG sudo greg
```

### Step 8: Copy Your SSH Key to the New User
```bash
mkdir -p /home/greg/.ssh
cp /root/.ssh/authorized_keys /home/greg/.ssh/
chown -R greg:greg /home/greg/.ssh
chmod 700 /home/greg/.ssh
chmod 600 /home/greg/.ssh/authorized_keys
```

### Step 9: Update Your Windows SSH Config
Back in PowerShell on your PC:
```powershell
notepad $env:USERPROFILE\.ssh\config
```

Change `User root` to `User greg` (keep the same port):
```
Host hostinger-vps
    HostName YOUR_IP
    User greg
    Port YOUR_PORT
    IdentityFile ~/.ssh/hostinger_vps
```
Save.

### Step 10: Test the New User

**Keep your current root session open.** Open a **second** PowerShell window:
```powershell
ssh hostinger-vps
```
You should see `greg@server:~$`. If this works, continue. If not, use the root session to fix it.

### Step 11: Disable Root Login
Back in your **root** session:
```bash
sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf 2>/dev/null
systemctl restart sshd
```

Now use `sudo` for anything that needs root power (e.g., `sudo apt install nginx`). When using Claude Code, tell it: "Use sudo when needed."

***

## Phase 3: Lock Down the Server

From here on, SSH in as greg: `ssh hostinger-vps`

### Step 12: Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 13: Harden SSH
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

> **WARNING: Do NOT close your current terminal until you test this.** If something goes wrong, you need the existing session to fix it. If you close both and can't connect, use the Hostinger web terminal (Dashboard → VPS → Overview → Terminal button).

```bash
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/*.conf 2>/dev/null
sudo systemctl restart sshd
```

**Test immediately:** Open a **new** PowerShell window → `ssh hostinger-vps`. If it works, you're good. If not, fix it in your existing session.

### Step 15: Firewall (UFW)
Use your SSH port from Step 1 (22 or 2222):
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit YOUR_PORT/tcp
sudo ufw enable
```
Type `y` to confirm.

```bash
sudo ufw status verbose
```
Should show: default deny incoming, SSH rate-limited.

> **Important: Hostinger has a SECOND firewall.** Go to hpanel.hostinger.com → VPS → Firewall. Both UFW (on the server) and the Hostinger firewall (in the dashboard) need to allow the same ports. If you open port 80 in UFW but not in Hostinger's dashboard, traffic won't reach your server.

### Step 16: Fail2Ban
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

This means: 3 failed SSH login attempts within 15 minutes = 4-hour ban.

### Step 17: Auto Security Updates
```bash
sudo apt install unattended-upgrades -y
```
Ubuntu auto-enables security patches once installed. Verify:
```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```
Should show both lines set to `"1"`. Done — security patches now install themselves.

### Step 18: Swap File
Claude Code + Node.js can spike past your RAM during builds. Without swap, Linux kills random processes. This prevents that.

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

***

## Phase 4: Tmux (Forever Sessions)

```bash
sudo apt install tmux -y
```

Create config:
```bash
cat > ~/.tmux.conf << 'EOF'
set -g mouse on
set -g history-limit 50000
EOF
```

Mouse mode lets you scroll. 50k line history means Claude Code's long outputs won't get cut off.

**Moves:**
| Action | Command |
|--------|---------|
| New session | `tmux new -s work` |
| Detach (leave running) | Ctrl+B, then D |
| Reattach | `tmux a -t work` |
| List sessions | `tmux ls` |
| Kill session | `tmux kill-session -t work` |

***

## Phase 5: Launch Claude Code

```powershell
ssh hostinger-vps
tmux new -s work
claude
```

Follow the auth URL it gives you (once). If `claude` isn't found, install manually:
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g @anthropic-ai/claude-code
claude
```

Test prompts: "What OS am I on?", "How much disk space is free?", "Show me ufw status"

***

## Phase 6: First Project

Claude prompt:
```
Create a landing page with "Coming Soon" as the headline and "Something awesome is on the way..." as the subheading. Serve it with Nginx on port 80. Test the config with nginx -t before reloading.
```

Then open the firewall:
```bash
sudo ufw allow 80/tcp
```

Also open port 80 in Hostinger dashboard (hpanel → VPS → Firewall).

Browser: `http://YOUR_IP`

### Pro tip: CLAUDE.md
Create a file called `CLAUDE.md` in any project folder. Claude Code reads it automatically at the start of every session. Use it for persistent instructions like "always use sudo when needed" or "this project uses Python 3.12." Saves you from repeating yourself.

***

## Phase 7: Test Forever Session

1. Detach: Ctrl+B, then D
2. Disconnect: `exit`
3. Close PowerShell entirely
4. Reopen PowerShell → `ssh hostinger-vps` → `tmux a -t work`
5. Claude Code should be right where you left it

***

## Daily Workflow

**Start:**
```powershell
ssh hostinger-vps
tmux a -t work
```
If no session exists: `tmux new -s work` then `claude`

**Stop:** Ctrl+B, then D → `exit`

***

## Quick Reference: What's Open

After completing this guide, your server allows:

| Port | Service | Status |
|------|---------|--------|
| YOUR_PORT | SSH | Rate-limited (ufw limit) |
| 80 | HTTP | Open (after Phase 6) |

Everything else is blocked. Open more ports only when Claude Code needs them (e.g., `sudo ufw allow 443/tcp` for HTTPS).

***

## Troubleshooting

| Issue | Fix |
|-------|-----|
| ssh-agent fails | Run PowerShell as Admin: `Get-Service ssh-agent \| Set-Service -StartupType Automatic; Start-Service ssh-agent` then regular PowerShell: `ssh-add $env:USERPROFILE\.ssh\hostinger_vps` |
| Permission denied (publickey) | Redo Step 3 or add key via Hostinger dashboard (Settings → SSH Keys). Web terminal fix: `chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys` |
| Connection refused | Check: correct port? UFW allows it? Hostinger firewall allows it? Windows firewall blocking outbound? |
| `claude` not found | Manual Node + Claude Code install (Phase 5) |
| tmux session gone | VPS probably rebooted. Start fresh: `tmux new -s work` then `claude` |
| Can't scroll in tmux | `tmux kill-session -t work` then recreate — the config file needs a fresh session to take effect |
| Locked out completely | Hostinger web terminal (Dashboard → VPS → Overview → Terminal button). Fix SSH config there. Root password reset available in dashboard if needed. |
| Locked out of greg, need root | Web terminal → `sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config && sudo systemctl restart sshd` → fix issue → re-disable root login |
| Website not loading | Did you open port 80 in **both** UFW and Hostinger dashboard firewall? |

***

## Maintenance (Monthly, 5 Min)

```bash
ssh hostinger-vps
sudo apt update && sudo apt upgrade -y
sudo fail2ban-client status sshd
sudo ufw status
```

Security patches auto-install, but regular updates catch everything else. Check fail2ban and UFW to make sure they're still running.

Take a Hostinger snapshot before any major system change (new software, config changes, etc.).

***

Guide complete. Questions → ask Claude Code on the VPS: it knows Linux better than both of us.
