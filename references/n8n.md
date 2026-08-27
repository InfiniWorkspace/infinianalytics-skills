# n8n (self-hosted or cloud)

There's no dedicated node. The workflow calls the REST API from HTTP Request nodes at each lifecycle stage. Read [rest-api.md](rest-api.md) for the endpoint, headers, and body; this file covers the n8n wiring.

Official starter template (InfiniAnalytics HTTP calls already wired around a Claude/Slack ticket-triage flow, ready to adapt):
`https://n8n.io/workflows/16929-triage-support-tickets-with-claude-sonnet-infini-analytics-and-slack/`

## Wiring

1. **Credential.** Create an HTTP header auth credential holding the `token` header with the organization token, reused across all the request nodes (keeps the token out of node parameters).

2. **Execution ID.** At the start of the workflow, add a Set node (name it `execution_id`) with field `execution_id` set to:

   ```
   {{ new Date().toISOString() }}
   ```

3. **Reference it in every later node.** Each subsequent HTTP Request node's body uses:

   ```
   {{ $('execution_id').first().json.execution_id }}
   ```

4. **Lifecycle HTTP Request nodes.** `POST https://api.analytics.infini.es/v1/register/` with the credential attached: `START` right after the Set node, `EVENT` after each meaningful step, `END` on the success branch.

5. **Error path.** Connect the workflow's Error Trigger (or the error output of failing nodes) to an `ERROR` request node, so a run that throws partway is still recorded as failed. Without this, crashed runs show as never finishing.
