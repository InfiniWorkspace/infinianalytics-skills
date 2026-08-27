# AI agents via MCP

The `@infini-analytics/analytics-mcp` npm package exposes one MCP tool, `register_event`, which sends a lifecycle event for the current agent session. Each conversation becomes a tracked execution in the dashboard. Requires Node.js 18+ (for `npx`).

## Install

**Claude Code:**

```bash
claude mcp add infini-analytics -e INFINI_TOKEN=your_token -- npx @infini-analytics/analytics-mcp
```

Verify with `claude mcp list` — `infini-analytics` should appear.

**VS Code (Copilot agent mode):**

```bash
npx @infini-analytics/analytics-mcp install
```

The installer registers the server in VS Code's MCP config and prompts for the token, saved as the `INFINI_TOKEN` env var. Restart VS Code afterwards.

## Configure the agent

1. **Config file** — create `.github/agents/infini-analytics.config.md` in the repository:

   ```markdown
   # Infini Analytics Configuration

   automation_id: YOUR_AUTOMATION_UUID
   ```

   Keeping the ID in a file (rather than in the prompt) lets several agents share one tracking block.

2. **Agent frontmatter** — declare the server in the agent prompt's tools list:

   ```yaml
   ---
   name: MyAgent
   tools: [..., infini-analytics]
   ---
   ```

3. **Tracking block** — append to the agent prompt body:

   ```markdown
   ---

   ## INFINI ANALYTICS LIFECYCLE TRACKING

   You must track every conversation run using the `register_event` tool
   from the `infini-analytics` MCP server.

   **Automation ID:** Read the value from `.github/agents/infini-analytics.config.md`
   at the start of every conversation.

   **Execution ID:** Generate once per conversation using an ISO 8601 timestamp
   at the moment the conversation starts: `YYYY-MM-DDTHH:MM:SS.ffffff`.

   ### Required lifecycle calls

   | When | Event | Description |
   |---|---|---|
   | Conversation starts (before any action) | START | "<Agent name> session started" |
   | Each meaningful step | EVENT | Brief description of the action |
   | After delivering the final response | END | "<Agent name> session completed" |
   | On unrecoverable failure | ERROR | Use error_id and error_description |

   **Rules:**
   - Always call START first, before reading any file or taking any action.
   - Always call END last, after your final message to the user.
   - On fatal error, call ERROR instead of END - never both.
   - Never skip START or the closing END/ERROR.
   ```

Package page: https://www.npmjs.com/package/@infini-analytics/analytics-mcp
