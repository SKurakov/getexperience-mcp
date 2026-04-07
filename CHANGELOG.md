# GetExperience MCP Server — Changelog

## v0.7.0 — 2026-04-07 — Streamable HTTP transport

**Breaking change:** transport protocol migrated from SSE to Streamable HTTP.

### What changed

| | Before (v0.6.x) | After (v0.7.0) |
|---|---|---|
| **Transport** | SSE (`@modelcontextprotocol/sdk` 0.5) | Streamable HTTP (SDK 1.29) |
| **Initialize** | `GET /mcp` → SSE stream with `sessionId` | `POST /mcp` with JSON-RPC `initialize` request |
| **Send message** | `POST /mcp?sessionId=<id>` | `POST /mcp` with `mcp-session-id: <id>` header |
| **Session ID** | Query parameter `?sessionId=` | HTTP header `mcp-session-id` |
| **Notifications** | Via same SSE stream | `GET /mcp` with `mcp-session-id` header |
| **Close session** | Close SSE connection | `DELETE /mcp` with `mcp-session-id` header |

### How to connect (v0.7.0)

```
1. POST /mcp
   Body: {"jsonrpc":"2.0","method":"initialize","params":{...},"id":1}
   Headers: Content-Type: application/json
            X-Api-Key: <key>          (optional, for B2B)
   Response: JSON-RPC result + mcp-session-id header

2. POST /mcp
   Body: {"jsonrpc":"2.0","method":"tools/call","params":{...},"id":2}
   Headers: Content-Type: application/json
            mcp-session-id: <id from step 1>

3. DELETE /mcp
   Headers: mcp-session-id: <id>
```

### What did NOT change

- All 6 tools: `search_experiences`, `get_experience_details`, `get_experience_schedule`, `get_checkout`, `add_to_checkout`, `create_order`
- Tool parameters and response formats
- `X-Api-Key` for B2B partners (passed in headers at initialize)
- `/health` endpoint
- Rate limiting (20 req/min anonymous, configurable per partner)
- Max 5 sessions per IP

### For SDK/client users

If you use the official MCP SDK (Python, TypeScript), update to SDK ≥1.29 and switch from `SSEClientTransport` to `StreamableHTTPClientTransport`. The SDK handles the protocol automatically.

### Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/mcp` | Initialize session or send JSON-RPC message |
| `GET` | `/mcp` | SSE stream for server notifications (requires `mcp-session-id`) |
| `DELETE` | `/mcp` | Terminate session (requires `mcp-session-id`) |
| `GET` | `/health` | Health check: `{"status":"ok","activeSessions":N}` |

---

## v0.6.5 — 2026-04-07

- Cancellation policy in `get_experience_details` (mandatory disclosure)
- Rate limiting per IP (20 req/min anonymous, configurable per partner key)
- Stripe Checkout Session for B2C payments (`paymentPageUrl`)
- Default cancellation policy when host hasn't set one

## v0.6.0 — 2026-03-13

- Simplified location search: single `location` param (free text, any language)
- Removed `query`, `seoSlug`, `googlePlaceId` params (breaking)
- Rewritten tool descriptions for better agent comprehension

## v0.5.0 — 2026-03-13

- USD pricing, `languages[]` filter, improved tool descriptions

## v0.4.0 — 2026-03-10

- `location` param replaces `seoSlug` (autocomplete-based resolution)

## v0.3.x — 2026-02

- Booking flow: `add_to_checkout`, `create_order`
- B2B internal payment with `X-Api-Key`
- `[GXP_STRUCTURED]` JSON blocks in responses

## v0.2.x — 2026-02

- SSE transport, Docker, Kubernetes Helm chart
- `search_experiences`, `get_experience_details`, `get_experience_schedule`
