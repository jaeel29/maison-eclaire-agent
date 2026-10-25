# Maison Éclaire — MCP server & agent skill

<img src="logo.png" width="96" alt="Maison Éclaire logo">

[Maison Éclaire](https://www.maisoneclaire.co) is a handmade candle, gift and candle-making supplies shop in Morocco.
This repository lets AI assistants (ChatGPT, Claude, Gemini, Copilot, Cursor, OpenClaw…) browse the shop through its public **MCP server**.

- **Remote MCP endpoint:** `https://www.maisoneclaire.co/mcp` (Streamable HTTP, no auth, read-only)
- **Official MCP Registry:** `co.maisoneclaire/shop`
- **Server card:** https://www.maisoneclaire.co/mcp/server-card

The site's source code is not in this repository; it only contains the agent skill and client configs.

## Tools

| Tool | What it does |
|---|---|
| `search_products` | Search candles, gifts and supplies (`query`, `category`, `lang`, `in_stock_only`, `max_price`, `limit`) |
| `get_product` | Product details, price in MAD, stock and page link (`slug`, `lang`) |
| `list_categories` | Shop categories (`lang`) |
| `get_shop_info` | Delivery, cash-on-delivery and return terms (`lang`) |
| `get_gift_ideas` | Gift suggestions by budget and occasion (`budget`, `occasion`, `lang`, `limit`) |

Languages: `fr`, `en`, `ar`. Prices are in Moroccan dirhams (MAD). Ordering happens on the website.

## Connect

**Claude Code / Cursor / any MCP client** — use `.mcp.json`:

```json
{ "mcpServers": { "maison-eclaire": { "type": "http", "url": "https://www.maisoneclaire.co/mcp" } } }
```

**Claude.ai / ChatGPT** — add a custom connector with URL `https://www.maisoneclaire.co/mcp`.

**Gemini CLI** — `gemini extensions install https://github.com/jaeel29/maison-eclaire-agent`

**Agent skill** — see [`skills/maison-eclaire/SKILL.md`](skills/maison-eclaire/SKILL.md) (OpenClaw / ClawHub, Claude skills, `npx skills add jaeel29/maison-eclaire-agent`).

## License

MIT
