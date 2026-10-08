# Give Claude Code Eyes Inside Your WSL2 Docker Containers

> Stop pasting Docker Compose logs into Claude Code. Give it SSH access into WSL2 so it reads container logs directly, the same trick used with a Kali VM.

Source: https://exitcode0.net/posts/claude-code-wsl2-docker-compose-logs/
Author: Tom Cocking (https://tomcocking.com)
Published: 2026-04-23
Updated: 2026-10-08
Tags: claude-code, wsl2, docker, ssh, windows-11, ai, devops


If you run Docker inside WSL2 on Windows 11, you've probably hit this wall: you open a Claude Code session in Windows Terminal, something goes wrong with a container, and Claude starts asking "can you paste the logs?". Every. Single. Time.

It breaks the flow completely. The whole point of an agentic coding tool is that it should be able to diagnose problems itself, not act as a fancy chat window that waits for you to do the investigation manually.

The fix is surprisingly elegant: give Claude Code SSH access into WSL2. Same pattern as setting up SSH to a remote VM - just pointed at `localhost:2222` instead of something on your LAN. Once it's wired up, Claude can pull compose logs, check container status, and tail output entirely on its own.

I've put together a PowerShell script that automates the entire setup. You can either run that and be done in a couple of minutes, or follow the manual steps below if you want to understand what's happening.

## The Automated Route

Grab the setup script from GitHub and run it from a standard Windows PowerShell terminal - not WSL, no admin rights needed:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
irm https://raw.githubusercontent.com/Tombo1001/awesome-claude-tricks/refs/heads/main/WSL2/setup-claude-wsl2-ssh.ps1 | iex
```

The script handles everything in one go: generates a dedicated SSH key pair, installs and configures OpenSSH in WSL2, copies the public key across, sets up passwordless sudo for the service, configures auto-start, and writes the `wsl2` host entry to your Windows SSH config. It uses `wsl -u root` internally for privileged steps so nothing prompts for a sudo password and nothing hangs. At the end it prints a ready-to-paste `CLAUDE.md` snippet.

If you want to understand what's happening under the hood, read on.

## Why SSH and Not Just `wsl --`

Your first thought might be to use WSL interoperability. From Windows you can run `wsl -d Ubuntu -- docker compose logs` and it works. Claude Code can use this too as a quick win - add something like this to your `CLAUDE.md`:

```markdown
## Docker / WSL2
Docker runs in WSL2 Ubuntu. To read compose logs from Windows:
`wsl -d Ubuntu -- bash -c "cd ~/projects/myapp && docker compose logs --tail=50"`
```

That's 30 seconds to set up and genuinely useful. But it's context-free - Claude needs the full path every time and can't run arbitrary diagnostic commands in your full Linux environment.

SSH is cleaner. Once Claude has an SSH target it can run anything inside WSL2 - your shell aliases, your PATH, anything that depends on the Linux environment being fully initialised. It also mirrors the pattern you'd use for any other machine you give Claude access to (a Kali VM on your LAN for pentesting, a staging server, etc.). Same mental model, same config, same trust boundary.

## Step 1 - Install and Configure OpenSSH in WSL2

Open your WSL2 Ubuntu terminal and install the SSH server:

```bash
sudo apt update
sudo apt install openssh-server -y
```

WSL2 shares port 22 with Windows if you have OpenSSH for Windows installed, so switch ports to avoid conflicts. Edit the daemon config:

```bash
sudo nano /etc/ssh/sshd_config
```

Find and update these lines, uncommenting them if needed:

```
Port 2222
ListenAddress 0.0.0.0
PasswordAuthentication no
PubkeyAuthentication yes
```

Save, then start and check the service:

```bash
sudo service ssh start
sudo service ssh status
```

## Step 2 - Set Up Passwordless SSH Keys

Generate a dedicated key pair on Windows. Open PowerShell (not WSL):

```powershell
ssh-keygen -t ed25519 -C "claude-code-wsl2" -f "$env:USERPROFILE\.ssh\id_ed25519_wsl2"
```

Using a dedicated key avoids interfering with any existing keys you have. Now copy the public key into WSL2:

```powershell
$pubKey = Get-Content "$env:USERPROFILE\.ssh\id_ed25519_wsl2.pub"
wsl -d Ubuntu -- bash -c "mkdir -p ~/.ssh && echo '$pubKey' >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

Test it:

```powershell
ssh -p 2222 -i "$env:USERPROFILE\.ssh\id_ed25519_wsl2" yourusername@localhost
```

If you get a shell you're in. Type `exit` and move on.

## Step 3 - Create a Named SSH Host

Add a host entry to `C:\Users\YourName\.ssh\config` so you can reference WSL2 by a friendly name:

```
Host wsl2
  HostName localhost
  Port 2222
  User yourusername
  IdentityFile ~/.ssh/id_ed25519_wsl2
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

One gotcha here: this file must be UTF-8. PowerShell's `Add-Content` and `Set-Content` write UTF-16 LE by default and OpenSSH on Windows silently fails to parse it - you'll get a cryptic "Connection to UNKNOWN port -1" error that's pretty hard to diagnose. Write the file in Notepad, VS Code, or use `[System.IO.File]::AppendAllText` with an explicit `UTF8Encoding` object. The setup script handles this automatically.

Confirm it's working:

```powershell
ssh wsl2 "docker ps"
```

If you see your running containers, Claude Code will too.

## Step 4 - Auto-Start SSH on WSL2 Launch

WSL2 doesn't start services automatically. Add this to your WSL2 `~/.bashrc`:

```bash
sudo service ssh start > /dev/null 2>&1
```

This needs a passwordless sudo rule to work without prompting. Add it via `sudo visudo`:

```
yourusername ALL=(ALL) NOPASSWD: /usr/sbin/service ssh *
```

Now every time you open a WSL2 terminal, SSH comes up automatically.

## Step 5 - Set Up Your CLAUDE.md

This is where it all clicks together. Claude Code reads your `CLAUDE.md` at the start of every session, so you can teach it how to use the WSL2 connection once and never explain it again.

You can have multiple CLAUDE.md files and Claude reads all of them - they layer on top of each other:

| Location | Scope |
|---|---|
| `~/.claude/CLAUDE.md` | Global - loads in every Claude Code session |
| `~/yourproject/CLAUDE.md` | Project - loads when you `claude .` from that folder |
| `~/yourproject/subfolder/CLAUDE.md` | Sub-directory - loads when Claude works in that folder |

For the WSL2 and Docker setup, put it in your **global** CLAUDE.md so it applies everywhere. You probably don't have one yet - create it:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude"
notepad "$env:USERPROFILE\.claude\CLAUDE.md"
```

Add a section like this:

```markdown
## Docker / WSL2 Environment

- Docker and Docker Compose run inside WSL2 Ubuntu, not on Windows directly.
- SSH access to WSL2 is available via `ssh wsl2` - use this for all Docker operations.
- Always check container logs proactively. Do not ask me to paste them.

### Useful commands

    ssh wsl2 "docker compose -f ~/YOUR_PROJECT/docker-compose.yml logs --tail=100"
    ssh wsl2 "docker compose -f ~/YOUR_PROJECT/docker-compose.yml logs --tail=100 YOUR_SERVICE"
    ssh wsl2 "docker ps -a"
    ssh wsl2 "docker compose -f ~/YOUR_PROJECT/docker-compose.yml restart YOUR_SERVICE"
```

The critical instruction is "always check logs proactively - do not ask me to paste them". Without it Claude defaults to asking. With it, Claude SSHs in and pulls them as part of its normal diagnostic flow.

Then in each project's own `CLAUDE.md`, add the specific compose file path and service names for that stack. The project file adds to the global one - Claude reads both every session.

## Step 6 - Add a Slash Command for Quick Access

For an on-demand log dump without needing to describe what you want, create `.claude/commands/compose-logs.md` in your project:

```markdown
---
allowed-tools: Bash(ssh *)
description: Fetch Docker Compose logs from WSL2 Ubuntu
argument-hint: [service-name]
---

SSH into WSL2 and retrieve Docker Compose logs for this project.

Run the following and analyse the output, highlighting any errors, warnings, or crash loops:

`ssh wsl2 'docker compose -f ~/projects/PROJECTNAME/docker-compose.yml logs --tail=150 $ARGUMENTS'`

Summarise what you find and suggest next steps if there are any obvious issues.
```

Replace `PROJECTNAME` with your actual project path. Inside a Claude Code session, `/project:compose-logs` gives you an immediate log dump and analysis, or `/project:compose-logs nginx` to target a specific service.

## What This Looks Like in Practice

I start Claude Code from my project folder on Windows:

```
claude .
```

I describe what I'm working on, something breaks in a container, and instead of Claude asking me to paste logs it just does this:

```
Let me check the container logs...
> ssh wsl2 'docker compose -f ~/projects/myapp/docker-compose.yml logs --tail=100'
[output appears directly in session]
```

Claude reads the output, identifies the problem - missing env variable, failed health check, port conflict - and proposes a fix. The copy-paste loop is completely gone.

## Troubleshooting

**SSH hangs or times out**
The service probably hasn't started yet. Open a WSL2 terminal (which triggers the `.bashrc` auto-start) and try again, or run `sudo service ssh start` directly.

**Permission denied (publickey)**
Check that `~/.ssh/authorized_keys` in WSL2 contains your Windows public key and that permissions are set correctly: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`.

**"Connection to UNKNOWN port -1" error**
Your `~/.ssh/config` on Windows is almost certainly encoded as UTF-16. Open it in VS Code or Notepad and re-save explicitly as UTF-8. OpenSSH silently fails to parse UTF-16 config files and the error message gives you absolutely no indication that's the problem.

**SSH works manually but not from Claude Code**
Claude Code runs in a Windows shell context, not WSL. Make sure `ssh wsl2` works from a standard PowerShell terminal - that's the environment Claude's bash tool runs in.

**Claude uses the wrong project path**
Paths in `CLAUDE.md` must be WSL2 paths, not Windows paths. Your WSL2 home (`~`) maps to `\\wsl$\Ubuntu\home\yourusername` on the Windows side, but always reference it as `~` or `/home/yourusername` in commands passed over SSH.

---

Once this is in place Claude becomes a genuinely useful debugging partner for containerised work rather than a tool that needs you to do its legwork. If you're already giving Claude SSH access to other machines, adding WSL2 to that is a natural extension - same pattern, same config, just pointed at localhost. And if you want Claude to see a real browser as well as container logs, [giving Claude Code a headful Chromium in Docker](https://exitcode0.net/posts/claude-code-browser-docker-playwright-mcp/) is the next step along the same path.

The full setup script is on GitHub: [/awesome-claude-tricks/blob/main/WSL2/setup-claude-wsl2-ssh.ps1](https://github.com/Tombo1001/awesome-claude-tricks/blob/main/WSL2/setup-claude-wsl2-ssh.ps1)


---
Markdown version of https://exitcode0.net/posts/claude-code-wsl2-docker-compose-logs/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
