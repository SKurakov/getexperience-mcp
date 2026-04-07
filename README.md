# GetExperience MCP Server

MCP server for [GetExperience.com](https://getexperience.com) — a marketplace of tours and unique travel experiences in 40+ countries.

Search, browse, and book real tours and activities directly from any AI assistant via the Model Context Protocol.

## Who is this for

- **AI assistants and chatbots** — add travel booking capabilities to your product
- **Travel agencies and concierge services** — automate tour search and booking through AI
- **Developers building AI agents** — plug in a ready-made travel API via MCP

### Free access (no API key)

Browse the full catalog: search experiences, view details, check schedules and pricing. Works out of the box — just connect to the endpoint. Instant payment and booking for anyone through the chat — coming soon.

### B2B partner integration

Book tours on behalf of your customers — instant confirmation, no guest payment required. Your company settles with GetExperience separately (invoice/balance).

**To get started as a B2B partner, contact us at [bookings@getexperience.com](mailto:bookings@getexperience.com) — we'll issue your API key the same day.**

## Server URL

```
https://getexperience.com/mcp
```

Transport: **Streamable HTTP** (since v0.7.0). No installation required — it's a hosted remote server.

> **Migrating from SSE?** See [CHANGELOG.md](CHANGELOG.md#v070--2026-04-07--streamable-http-transport) for the full before/after comparison.

## Quick Start

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "getexperience": {
      "type": "streamablehttp",
      "url": "https://getexperience.com/mcp"
    }
  }
}
```

### Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "getexperience": {
      "type": "streamablehttp",
      "url": "https://getexperience.com/mcp"
    }
  }
}
```

### Any MCP Client

Point your MCP client to:
```
https://getexperience.com/mcp
```

No API key required for browsing. For B2B booking, add `X-Api-Key` header:

```json
{
  "mcpServers": {
    "getexperience": {
      "type": "streamablehttp",
      "url": "https://getexperience.com/mcp",
      "headers": {
        "X-Api-Key": "YOUR_API_KEY"
      }
    }
  }
}
```

## Tools

| Tool | Description |
|------|-------------|
| `search_experiences` | Search tours by location, category, date, price, guests, language |
| `get_experience_details` | Full details: description, photos, pricing, duration, languages, rating |
| `get_experience_schedule` | Available time slots and pricing for a specific date |
| `add_to_checkout` | Add an experience to the cart |
| `get_checkout` | View current cart contents |
| `create_order` | Place the final booking (B2B instant or B2C via Stripe) |

Full parameters, response schemas, and examples: **[TOOLS.md](TOOLS.md)**

## Booking Flow

```
search_experiences → get_experience_schedule → add_to_checkout → create_order
```

1. **Search** — find experiences by location, category, dates
2. **Schedule** — pick a date, get available time slots with prices
3. **Add to cart** — choose a slot, specify guests
4. **Order** — place the booking (B2C via Stripe, or B2B with API key)

### B2C (Guest pays by card)

Set `paymentSystem: "stripe"` in `create_order`. Returns a Stripe payment link — the guest completes payment there. *(Coming soon)*

### B2B (Partner integration)

Set `paymentSystem: "internal"` and provide `X-Api-Key` header at connection time. Booking is confirmed instantly, host is notified by email.

Contact [bookings@getexperience.com](mailto:bookings@getexperience.com) to get your API key.

## Response Format

All tools support two response formats via the `responseFormat` parameter:

- **`"markdown"`** (default) — Human-readable text + `[GXP_STRUCTURED]` JSON block for programmatic use
- **`"json"`** — Pure JSON only

### Structured Data

When using markdown format, every response includes a `[GXP_STRUCTURED]` block with typed JSON:

```javascript
const match = text.match(/\[GXP_STRUCTURED\]([\s\S]*?)\[\/GXP_STRUCTURED\]/);
const data = match ? JSON.parse(match[1].trim()) : null;
```

## Health Check

```
GET https://getexperience.com/mcp/health
```

## Links

- **Website:** [getexperience.com](https://getexperience.com)
- **MCP Endpoint:** `https://getexperience.com/mcp`
- **Contact:** [bookings@getexperience.com](mailto:bookings@getexperience.com)

## License

MIT
