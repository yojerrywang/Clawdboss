# Recommended Tools

Clawdboss includes integration guides for vetted OpenClaw ecosystem tools. These are optional but recommended.

## Clawmetry — Observability Dashboard

**What:** Real-time monitoring dashboard showing token costs, sessions, cron jobs, sub-agents, memory, and live message flow. Zero config — auto-detects everything.

**Install:**
```bash
pip install clawmetry
clawmetry
# Opens at http://localhost:8900
```

**What you get:**
- **Flow** — Live animated diagram of messages through channels → brain → tools → response
- **Usage** — Token and cost tracking with daily/weekly/monthly breakdowns
- **Sessions** — Active agent sessions with model, tokens, last activity
- **Crons** — Scheduled jobs with status, next run, duration
- **Logs** — Color-coded real-time log streaming
- **Memory** — Browse SOUL.md, MEMORY.md, AGENTS.md, daily notes
- **Transcripts** — Chat-bubble UI for reading session histories

**Configuration (optional):**
```bash
clawmetry --port 9000           # Custom port (default: 8900)
clawmetry --host 127.0.0.1      # Bind to localhost only
clawmetry --workspace ~/mybot   # Custom workspace path
```

**Links:**
- GitHub: <https://github.com/vivekchand/clawmetry>
- Website: <https://clawmetry.com>
- License: MIT (free, open-source)

---

## ClawSec — Security Suite

**What:** Complete security skill suite from [Prompt Security](https://prompt.security). Provides drift detection for agent files (SOUL.md, AGENTS.md), advisory feed monitoring, and malicious skill detection.

**Install:**
```bash
# Option A: Via clawhub
npx clawhub@latest install clawsec-suite

# Option B: Manual from GitHub
git clone --depth 1 https://github.com/prompt-security/clawsec.git /tmp/clawsec
cp -r /tmp/clawsec/skills/clawsec-suite ~/.openclaw/skills/
cp -r /tmp/clawsec/skills/soul-guardian ~/.openclaw/skills/
cp -r /tmp/clawsec/skills/clawsec-feed ~/.openclaw/skills/
rm -rf /tmp/clawsec
```

### Soul Guardian — File Integrity Protection

Detects unauthorized changes to SOUL.md, AGENTS.md, IDENTITY.md and auto-restores critical files.

```bash
# Initialize baselines (run once after setup)
cd ~/.openclaw/workspace
python3 ~/.openclaw/skills/soul-guardian/scripts/soul_guardian.py init --actor setup --note "initial baseline"

# Test the check
python3 ~/.openclaw/skills/soul-guardian/scripts/soul_guardian.py check --actor test --output-format alert
```

### Advisory Feed — Vulnerability Monitoring

Polls the ClawSec advisory feed for CVEs and malicious skill reports. Cross-references against your installed skills.

Add to your `HEARTBEAT.md` for automatic monitoring:
```markdown
## Soul Guardian Check
- Run `python3 ~/.openclaw/skills/soul-guardian/scripts/soul_guardian.py check --actor heartbeat --output-format alert`
- If any output is produced, relay it as a security alert

## ClawSec Advisory Feed (weekly)
- Check ~/.openclaw/clawsec-suite-feed-state.json for last check time
- Run advisory feed check from ~/.openclaw/skills/clawsec-suite/HEARTBEAT.md
- Report any new advisories affecting installed skills
```

**Links:**
- GitHub: <https://github.com/prompt-security/clawsec>
- Website: <https://clawsec.prompt.security>
- License: MIT (free, open-source)
- Requires: python3, curl (jq optional)
