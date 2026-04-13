# skill-auditor-in-sandbox

A Claude Code skill that launches a NovitaClaw (OpenClaw) sandbox, installs a specified skill, and generates an installation & security audit report.

## Project Structure

```
skill-auditor-in-sandbox/
├── skill-auditor-in-sandbox.md           # Skill definition (dispatcher + report template)
├── scripts/
│   ├── install-skill.mjs              # Connects to sandbox, installs skill, outputs JSON
│   └── audit-skill.mjs               # Runs security audit in sandbox, outputs JSON
├── package.json
├── .gitignore
├── README.md
└── LICENSE
```

The skill file is a lightweight dispatcher. The heavy lifting (sandbox SDK calls, fallback logic, pattern scanning) lives in `scripts/`, which output structured JSON for the skill to parse.

## Installation

Clone this repo and use it directly:

```bash
git clone https://github.com/NovitaAI/skill-auditor-in-sandbox.git
cd skill-auditor-in-sandbox
npm install
```

Or copy into an existing project:

```bash
cp skill-auditor-in-sandbox.md /path/to/project/.claude/skills/
cp -r scripts/ /path/to/project/scripts/
```

## Prerequisites

- **Node.js** >= 18.0.0 (required for top-level await and ESM support)
- **NOVITA_API_KEY** — set in your environment. Get one from [Novita AI Key Management](https://novita.ai/settings/key-management).
- **novitaclaw CLI** — install with:
  ```bash
  curl -fsSL https://novitaclaw.novita.ai/install.sh | bash
  ```

## Usage

In Claude Code, invoke the skill with:

```
/skill-auditor-in-sandbox <skill-name>
```

Example:

```
/skill-auditor-in-sandbox pskoett/self-improving-agent
```

### What it does

1. Launches a NovitaClaw sandbox (`novitaclaw launch`)
2. Installs the skill via `scripts/install-skill.mjs` (tries clawhub -> GitHub -> clawhub.ai git clone)
3. Runs a security audit via `scripts/audit-skill.mjs`
4. Generates a structured report with risk assessment

### Scripts (standalone usage)

The scripts can also be run independently:

```bash
# Install a skill into a sandbox
SANDBOX_ID=<id> NOVITA_API_KEY=<key> SKILL_NAME=pskoett/self-improving-agent node scripts/install-skill.mjs

# Audit an installed skill
SANDBOX_ID=<id> NOVITA_API_KEY=<key> SKILL_NAME=pskoett/self-improving-agent node scripts/audit-skill.mjs
```

Both scripts output JSON to stdout.

## Security Audit

The audit script checks for:

- Suspicious code patterns (subprocess, eval, exec, base64, crypto mining, etc.)
- Network calls (all URLs referenced in skill files)
- File system access outside the skill directory (/etc/, /root/, ~/.ssh, /tmp/)
- External dependencies (requirements.txt, package.json)
- Full file contents for manual review

## License

MIT
