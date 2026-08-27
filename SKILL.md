---
name: infinianalytics-skills
description: Wire InfiniAnalytics execution tracking into an automation (Python, REST, Power Automate Desktop/Cloud, n8n, or an AI agent via MCP). Use when the user wants to track, instrument, or monitor an automation with InfiniAnalytics, add lifecycle events (START/EVENT/WARNING/END/ERROR), or debug executions that don't appear in the dashboard.
---

# InfiniAnalytics automation tracking

Wire InfiniAnalytics execution tracking into a user's automation so every run shows up in the dashboard as an execution with a full event timeline. This skill covers the user-facing integration only — sending events from the automation — not InfiniAnalytics platform development.

## The lifecycle contract

Every integration, whatever the platform, sends the same five event types for each run:

| Event | When | Cardinality |
|---|---|---|
| `START` | The very first thing the run does | Exactly one per run |
| `EVENT` | Each meaningful milestone ("File downloaded", "50 rows processed") | As many as useful |
| `WARNING` | A non-critical issue worth reviewing, without failing the run | As needed |
| `END` | Successful completion, the last thing the run does | One — or `ERROR`, never both |
| `ERROR` | Failure, from the error handler; marks the execution as failed | Replaces `END` |

A well-formed run is `START` → (`EVENT`/`WARNING`)* → `END` or `ERROR`. InfiniAnalytics takes the `START` timestamp as the execution start, the `END`/`ERROR` timestamp as the end, and computes duration and outcome from them. Events sent after `END`/`ERROR` are accepted but do not change the outcome.

**The execution ID is the correlation key.** The user's side generates it (a UUID v4 or an ISO 8601 timestamp both work) — InfiniAnalytics never assigns one. Every event of the same run must carry the same ID; all events sharing an ID are stitched into one execution record. It must be unique per run: reusing an ID across runs merges them into one record, and it must not collide with a concurrent run. Multi-technology workflows (e.g. a Power Automate flow calling a Python script) may share one ID across technologies to appear as a single execution.

**Two credentials, both from the InfiniAnalytics dashboard:**

- **Organization token** — on the organization page (`/organizacion`). One token for the whole organization, all integrations, no expiry. It grants write access to every automation, so treat it as a secret: environment variable or platform secret store, never hardcoded in source, config committed to git, or client-side code.
- **Automation UUID** — in the header of the automation's detail page. Routes events to the right automation.

## Steps

1. **Identify the platform.** Look at the files or description the user gave: a Python script, a generic script/service in another language, a Power Automate Desktop or Cloud flow, an n8n workflow, or an AI coding agent. If nothing indicates the platform, ask.

2. **Confirm the automation exists in the dashboard.** The user needs an automation UUID. If they don't have one: in the dashboard, create a **Process** (Processes → New Process: name, department, expected executions/year, time saved per execution — the last two drive the ROI figures, so have the user estimate them honestly), then inside it a **New Automation**, then copy the UUID from the automation detail header. Get the organization token from the organization page. These are dashboard actions only the user can do — give them the exact clicks and wait for the two values (the token can stay a placeholder like `INFINI_TOKEN` if they prefer not to paste it).

3. **Read the platform reference and integrate.** Load exactly the file for the platform in hand:

   - Python script, agent, or backend service → [references/python.md](references/python.md) (official `infinianalytics` PyPI package)
   - Any other language, or anything that can make an HTTP request → [references/rest-api.md](references/rest-api.md)
   - Power Automate Desktop or Cloud → [references/power-automate.md](references/power-automate.md) (official templates, no code)
   - n8n workflow → [references/n8n.md](references/n8n.md)
   - AI coding agent (Claude Code, VS Code Copilot agent mode) → [references/mcp.md](references/mcp.md) (MCP server)

   Then place the calls so that **every exit path of the automation emits `END` or `ERROR`** — the happy path ends with `END`, and each error handler (except block, error branch, error scope, global handler) emits `ERROR` with `error_id` and `error_description` filled from the caught failure. Generate the execution ID once at the top of the run and thread it through every call.

4. **Verify.** The integration is done when a real (or test) run produces a complete execution in the dashboard:
   - Trigger one run end to end (or, for flows the user must run by hand, have them trigger it).
   - Confirm the API accepted the events — `POST /v1/register/` returns HTTP 201 per event (the SDK and templates handle this silently; for REST/n8n check the response).
   - Have the user open the automation's detail page: one new execution, with the START/…/END timeline and a success outcome. Until that execution is visible, the task is not done — debug below.

## Debugging: events not appearing

- **401 Unauthorized** — `token` header missing or wrong. It is a plain `token:` header, not `Authorization: Bearer`.
- **301 Moved Permanently** — the request went over plain HTTP and was redirected instead of processed. Use `https://` explicitly.
- **Events land in the wrong execution / runs merged together** — execution ID reused across runs. Generate a fresh one per run.
- **Run shows as never finishing** — an exit path emits no `END`/`ERROR` (usually a crash before the error handler, or an error branch not wired). Re-check step 3's every-exit-path rule.
- **Python: nothing sent, no exception** — the SDK never raises; on failure it prints and returns `None`. Check the script's stdout for its messages.
- **Connectivity check** — `POST https://api.analytics.infini.es/v1/ping/` with the `token` header confirms token and network before touching the real integration.
