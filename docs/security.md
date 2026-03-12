# Security Architecture

## Secret Management

Clawdboss separates secrets from configuration:

### How It Works

1. **`.env` file** (`~/.openclaw/.env`) — Contains all API keys and tokens
   - Created during setup with `600` permissions (owner-only read/write)
   - Listed in `.gitignore` — never committed
   - Template provided as `.env.example`

2. **`openclaw.json`** — Contains `${VAR_NAME}` references instead of raw keys
   - OpenClaw resolves these at startup from the `.env` file
   - Config can be safely reviewed, diffed, and versioned
   - No secrets in the JSON

### Precedence Order

OpenClaw loads env vars in this order (first wins):
1. Process environment (parent shell)
2. `.env` in current working directory
3. `~/.openclaw/.env` (global)
4. `env` block in `openclaw.json`

### Example

**`.env`:**
```
DISCORD_BOT_TOKEN=MTQ3NjA4...actual-token
BRAVE_API_KEY=BSAab7lL...actual-key
```

**`openclaw.json`:**
```json
{
  "channels": {
    "discord": {
      "token": "${DISCORD_BOT_TOKEN}"
    }
  },
  "tools": {
    "web": {
      "search": {
        "apiKey": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

## Prompt Injection Defense

All agents are pre-configured with defense rules in their `AGENTS.md`:

### Rules Applied to Every Agent

1. **Content isolation** — External content (email, web, RSS) is treated as DATA, never INSTRUCTIONS
2. **Pattern detection** — Known attack patterns flagged and reported
3. **Anti-loop protection** — 5 rules preventing token-burning attacks
4. **Memory protection** — Agents cannot modify their own SOUL.md or AGENTS.md based on external content
5. **Email safety** — Human approval required for all external sends, draft-only mode

### Known Attack Vectors Defended

| Vector | Defense |
|--------|---------|
| Hidden HTML instructions in emails | HTML stripping + content tagging |
| "Ignore previous instructions" | Pattern detection + flagging |
| WORKFLOW_AUTO.md payload | Explicit warning in all AGENTS.md |
| Spoofed system messages | Real OpenClaw system messages include sessionId |
| Memory poisoning | Agents can't modify core files from external content |

## Context Persistence & Recovery

Clawdboss includes the **WAL (Write-Ahead Log) Protocol** to protect against context loss:

### How It Works

1. **SESSION-STATE.md** — Active working memory. Every correction, decision, and important detail is written here BEFORE the agent responds. This is the agent's "RAM" — chat history is just a buffer.

2. **Working Buffer** (`memory/working-buffer.md`) — When context usage hits ~60%, agents log every exchange to this file. After compaction (context window reset), the agent reads this buffer first to recover what was discussed.

3. **Compaction Recovery** — When an agent detects context loss (session starts with `<summary>`, or knowledge gaps), it automatically reads the working buffer and SESSION-STATE.md to recover — never asks "what were we discussing?"

### Why This Matters

Without WAL, agents lose corrections, decisions, and details when context compacts. A user says "it's blue, not red" — the agent acknowledges it — context compacts — the agent goes back to red. WAL prevents this by writing the correction to a file before responding.

## Network Security

- OpenClaw Gateway binds to `127.0.0.1` (localhost only)
- Dashboard should bind to `127.0.0.1` (not `0.0.0.0`)
- API keys have `600` file permissions
- Discord exec approvals enabled by default

## Brute-Force Protection (fail2ban)

The setup wizard offers to install and configure [fail2ban](https://github.com/fail2ban/fail2ban) — an intrusion prevention framework that monitors log files and automatically bans IPs showing malicious behavior.

### What It Does

- **Monitors** SSH, web server, and other service logs for repeated authentication failures
- **Bans** offending IPs via firewall rules (iptables/nftables) for a configurable duration
- **Protects** against brute-force password attacks, credential stuffing, and automated scanning

### Default Configuration

The setup wizard creates `/etc/fail2ban/jail.local` with sensible defaults:

```ini
[DEFAULT]
bantime  = 1h        # Ban duration
findtime = 10m       # Window for counting failures
maxretry = 5         # Failures before ban

[sshd]
enabled = true       # Protect SSH by default
```

### Manual Installation

```bash
# Debian/Ubuntu
sudo apt-get install fail2ban

# Fedora/RHEL
sudo dnf install fail2ban

# Arch
sudo pacman -S fail2ban

# Enable and start
sudo systemctl enable --now fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### Adding More Jails

Protect additional services by adding jails to `/etc/fail2ban/jail.local`:

```ini
# Nginx authentication failures
[nginx-http-auth]
enabled = true

# Repeated 404s (scanners)
[nginx-botsearch]
enabled = true
```

### Why It Matters for OpenClaw Deployments

If you're running OpenClaw on a VPS (e.g., Contabo, Hetzner, DigitalOcean), your SSH port is internet-facing. Without fail2ban, attackers can attempt thousands of password combinations per hour. fail2ban stops this by banning IPs after a few failures — it's one of the most effective low-effort security measures for any Linux server.

## Recommendations

1. **Rotate API keys** on a schedule (quarterly minimum)
2. **Enable UFW** if running on a VPS: `sudo ufw allow ssh && sudo ufw enable`
3. **Install fail2ban** for brute-force protection (prompted during setup)
4. **Use Tailscale** for remote access instead of exposing ports
5. **Review the Ansible playbook** for VPS deployments: [openclaw-ansible](https://github.com/openclaw/openclaw-ansible)
