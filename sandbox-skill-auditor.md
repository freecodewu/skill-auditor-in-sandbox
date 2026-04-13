---
name: sandbox-skill-auditor
description: >
  Launch a NovitaClaw (OpenClaw) sandbox, install a specified skill,
  and generate an installation & security report.
  Use when the user wants to test an OpenClaw/NovitaClaw skill in an isolated sandbox environment.
argument-hint: "<skill-name>"
---

# Sandbox Skill Auditor

You are given a skill name (or identifier) as `$ARGUMENTS`. Your job is to launch a sandbox, install the skill, run a security audit, and generate a report.

## Prerequisites

- `NOVITA_API_KEY` must be set. If not, ask the user:
  `export NOVITA_API_KEY=<key>` (get one from https://novita.ai/settings/key-management)
- `novitaclaw` CLI must be installed. If not found:
  `curl -fsSL https://novitaclaw.novita.ai/install.sh | bash`
- `novita-sandbox` npm package must be available. If not:
  `npm install novita-sandbox`

## Step 1: Launch Sandbox

```bash
novitaclaw launch --json
```

Parse the JSON output and extract `sandbox_id` and `webui`. Save these for the report.

If launch fails, check `error_code` and `remediation` fields:
- `MISSING_API_KEY` → ask user for API key
- `SANDBOX_TIMEOUT` → retry with `--timeout 300`

## Step 2: Install Skill

Run the install script from the project root:

```bash
SANDBOX_ID=<sandbox_id> SKILL_NAME="$ARGUMENTS" node scripts/install-skill.mjs
```

The script outputs JSON: `{ success, method, skillDir, files, error? }`.
- If `success` is false, show the error and stop.
- Note the `method` used (clawhub / git-github / git-clawhub) for the report.

## Step 3: Security Audit

Run the audit script:

```bash
SANDBOX_ID=<sandbox_id> SKILL_NAME="$ARGUMENTS" node scripts/audit-skill.mjs
```

The script outputs JSON:
- `suspicious[]` — lines matching risky code patterns (eval, exec, subprocess, base64, etc.)
- `urls[]` — all URL references found in skill files
- `externalPaths[]` — references to paths outside the skill directory (/etc/, /root/, ~/.ssh, /tmp/)
- `dependencies` — contents of requirements.txt or package.json if present
- `fileContents[]` — full contents of all text files for manual review

## Step 4: Assess Risk

Based on audit results, assign a risk level:

| Risk Level | Criteria |
|------------|----------|
| LOW | No suspicious patterns, URLs are legitimate (GitHub, docs), no external paths |
| MEDIUM | Some suspicious patterns but explainable (e.g., `fetch()` for legitimate API calls) |
| HIGH | Unexplained network calls, access to sensitive paths, obfuscated code |
| CRITICAL | Credential exfiltration, crypto mining indicators, shell injection patterns |

## Step 5: Generate Report

Output a structured report:

```
## Skill Installation Report

**Skill:** <skill-name>
**Sandbox ID:** <sandbox_id>
**Web UI:** <webui_url>
**Timestamp:** <current time>

### Installation Status
- **Result:** SUCCESS / FAILED
- **Method:** <clawhub / git-github / git-clawhub>
- **Files Installed:** <count> files

### Installed Files
<table of files and their purpose>

### Security Analysis
- **Risk Level:** LOW / MEDIUM / HIGH / CRITICAL

### Suspicious Patterns Found
| File | Line | Pattern | Severity |
|------|------|---------|----------|
(or "None found")

### URL References
| File | URL | Context |
|------|-----|---------|
(list all URLs and whether they look legitimate)

### External Path References
(list any, or "None found")

### Dependencies
(list any, or "No external dependencies")

### Recommendations
- <recommendation based on findings>

### Sandbox Management
- To access: <webui_url>
- To pause (save costs): `novitaclaw pause <sandbox_id>`
- To stop (permanent): `novitaclaw stop <sandbox_id>`
```

After the report, ask the user if they want to keep the sandbox running, pause it, or stop it.

## Important Notes

- Always use `--json` flag with novitaclaw commands.
- The sandbox auto-terminates based on `keep_alive`. Suggest pause to save costs.
- Prefer `pause` over `stop` — stop is irreversible. Confirm before stopping.
