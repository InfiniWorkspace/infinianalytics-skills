# infinianalytics-skills

A [Claude Code](https://claude.com/claude-code) skill that instruments an automation with [InfiniAnalytics](https://analytics.infini.es) execution tracking, so every run shows up in the dashboard as a full START → EVENT → END/ERROR timeline. Point Claude Code at a script, Power Automate flow, n8n workflow, or AI agent and ask it to add tracking; the skill supplies the lifecycle contract and the platform-specific wiring so it gets done right the first time, without hand-reading the API docs.

## What's in here

- **`SKILL.md`** is the core playbook: the five-event lifecycle contract, the execution-ID rules, where to get the two credentials (organization token + automation UUID), the integration steps, and a debugging section for events that don't show up.
- **`references/`** has one file per platform, loaded only when relevant:
  - `python.md`, the official `infinianalytics` PyPI package
  - `rest-api.md`, the raw `POST /v1/register/` HTTP contract (any language)
  - `power-automate.md`, the official PAD/PAC templates
  - `n8n.md`, HTTP Request node wiring for n8n workflows
  - `mcp.md`, the MCP server for AI coding agents (Claude Code, VS Code Copilot)

## Install

Clone this repo directly into a folder Claude Code scans for skills. The destination folder name must stay `infinianalytics-skills`; it has to match the `name:` declared in `SKILL.md`.

**Project-only** (available, and shareable via git, just in that repo):

```bash
git clone https://github.com/InfiniWorkspace/infinianalytics-skills.git .claude/skills/infinianalytics-skills
```

**Global** (available in every project on this machine):

```bash
git clone https://github.com/InfiniWorkspace/infinianalytics-skills.git "$HOME/.claude/skills/infinianalytics-skills"
```

Restart Claude Code (or open a new session) afterwards so it picks up the new skill.

## Use

Ask naturally, for example "add InfiniAnalytics tracking to this script", or invoke it explicitly with `/infinianalytics-skills`. Claude will ask for your organization token (from the InfiniAnalytics `/organizacion` page) and the automation's UUID, guiding you through creating a Process and Automation first if neither exists yet.

## Update

```bash
cd .claude/skills/infinianalytics-skills
git pull
```

## Contract source

The lifecycle contract and platform details here are kept in sync with the InfiniAnalytics `/docs` site. See the **Claude Skills** section there for the same install instructions in the product docs.
