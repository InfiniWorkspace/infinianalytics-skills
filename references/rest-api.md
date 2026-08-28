# REST API

Single endpoint for all five event types, callable from any language or platform that can make an HTTP request.

- Base URL: `https://api.analytics.infini.es`. HTTPS only; plain HTTP gets a 301 redirect instead of being processed.
- `POST /v1/register/` registers one event. The only endpoint an integration needs.
- `POST /v1/ping/` is a lightweight connectivity/token check.

## Headers

```http
token: your_organization_token
Content-Type: application/json
```

Authentication is this plain `token` header, not `Authorization: Bearer`, no OAuth, no expiry. Missing/invalid token → 401.

## Request body

| Field | Required | Notes |
|---|---|---|
| `automation_id` | Yes | Automation UUID from the dashboard |
| `execution_id` | Yes | Caller-generated run ID, identical across all events of the run |
| `event` | Yes | `START`, `EVENT`, `WARNING`, `END`, or `ERROR` |
| `description` | No | Free text describing the event; for `ERROR`, a human-readable explanation of the error and its likely cause |
| `error_id` | No | Only meaningful with `ERROR`. A short, searchable identifier for that error case, e.g. `ERR-01`, `ERR-02`, or an existing error code/exception name from the code |
| `error_description` | No | Only meaningful with `ERROR`. Exhaustive detail: the programming language's own exception message/stack trace when there is one, otherwise as much diagnostic detail as available |

## Example

```bash
curl --location 'https://api.analytics.infini.es/v1/register/' \
  --header 'Content-Type: application/json' \
  --header 'token: your_organization_token' \
  --data '{
    "automation_id": "3571c1e0-88b9-4104-a0b0-f060b7fcc6eb",
    "execution_id": "2025-03-03T15:32:17.571404",
    "event": "START",
    "description": "Starting the execution"
  }'
```

For `ERROR`, add `"error_id": "e0001", "error_description": "Detailed error description"`.

Success response is **HTTP 201** with the stored event echoed back:

```json
{
  "automation": "3571c1e0-88b9-4104-a0b0-f060b7fcc6eb",
  "execution_id": "2025-03-03T15:32:17.571404",
  "type_of_event": "START",
  "description": "Starting the execution",
  "error_id": "",
  "error_description_detail": "",
  "created_at": "2025-03-03T16:00:29.998530Z"
}
```

## Status codes

| Status | Meaning |
|---|---|
| 201 | Event registered |
| 301 | Request used plain HTTP; retry over HTTPS |
| 401 | `token` header missing or invalid |
| 5xx | Transient server issue, retry later; contact Infini support if it persists |
