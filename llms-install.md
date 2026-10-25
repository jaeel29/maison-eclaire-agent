# Installing the Maison Éclaire MCP server (for AI agents such as Cline)

Maison Éclaire's MCP server is **remote and hosted**. There is nothing to clone, build or install, and **no API key or authentication** is needed.

- Endpoint: `https://www.maisoneclaire.co/mcp`
- Transport: Streamable HTTP
- Access: read-only (catalogue, gift ideas, delivery terms). It never places orders or takes payments.

## Cline

Add this entry under `mcpServers` in `cline_mcp_settings.json`. Set `type` to `streamableHttp` explicitly, because if it is left out Cline falls back to legacy SSE:

```json
{
  "mcpServers": {
    "maison-eclaire": {
      "type": "streamableHttp",
      "url": "https://www.maisoneclaire.co/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Or, in Cline go to **MCP Servers → Remote Servers**, then set Server Name to `maison-eclaire`, Server URL to `https://www.maisoneclaire.co/mcp`, and Transport Type to **Streamable HTTP**.

## Verify

After connecting, these five tools should be listed: `search_products`, `get_product`, `list_categories`, `get_shop_info` and `get_gift_ideas`. You can test with `get_shop_info` (no arguments) or `search_products` with the query `"bougie"`.

## Usage notes

- Prices are in MAD (Moroccan dirham). Product content is available in French, English and Arabic.
- For guidance on how to answer shoppers, see `skills/maison-eclaire/SKILL.md`.
