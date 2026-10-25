---
name: maison-eclaire
description: Shop Maison Éclaire, a Moroccan maker of handmade scented candles, gift boxes, flower boxes, candle holders and wedding favours, and a supplier of candle-making supplies (wax, wicks, fragrance oils, dyes, moulds, kits). Use when someone wants to find or compare candles or gifts in Morocco, pick a gift for a budget or occasion (birthday, wedding, Ramadan, Eid, Valentine's), buy candle-making supplies in Morocco, or check prices in MAD, delivery fees, cash on delivery or returns. Read-only public MCP server, no API key.
version: 1.0.0
metadata:
  openclaw:
    emoji: "🕯️"
    homepage: https://www.maisoneclaire.co
---

# Maison Éclaire

Maison Éclaire (https://www.maisoneclaire.co) is a Moroccan shop for handmade candles, gifts and
candle-making supplies. It has a public, **read-only** MCP server that you can use to search the
catalogue, read product details, suggest gifts and quote the shop's delivery and return terms.

- Endpoint: `https://www.maisoneclaire.co/mcp`
- Transport: Streamable HTTP (stateless; no session id)
- Auth: none. No API key, no account, no environment variables.
- Registry name: `co.maisoneclaire/shop` (official MCP registry)

## Rules for answering

1. **Always link the product.** Every product the tools return has a `url`. Give it to the user,
   because that is where they order. Do not make up URLs or slugs.
2. **Prices are in Moroccan dirhams (MAD).** Show them as "290 MAD". If `price_is_from` is true,
   say "from 399 MAD" (the price depends on the variant). If `old_price_mad` is set, the item is
   on sale.
3. **Out-of-stock products have no price.** Say it is sold out; don't guess a price.
4. **The tools cannot order, pay or change anything.** To buy, the user opens the product URL and
   orders on the website (no account needed). Payment is **cash on delivery only**. No card is
   ever asked for, so never ask the user for card details.
5. **Get delivery and return terms from `get_shop_info`.** Don't restate them from memory; they
   can change. For anything the tools don't cover, send the user to the contact page
   (`/en/contactez-nous`).
6. Answer in the user's language. The tools accept `lang`: `fr` (default), `en` or `ar`. Queries
   can be in any language.

## Connect

Claude Code:

```bash
claude mcp add --transport http maison-eclaire https://www.maisoneclaire.co/mcp
```

Any client that reads an `mcpServers` config (Cursor, VS Code, Windsurf, Claude Desktop through a
remote connector, and others):

```json
{
  "mcpServers": {
    "maison-eclaire": {
      "url": "https://www.maisoneclaire.co/mcp"
    }
  }
}
```

In ChatGPT or Claude, add it as a custom connector with the URL `https://www.maisoneclaire.co/mcp`
and no authentication.

If your agent cannot add MCP servers, it can call the endpoint directly with JSON-RPC over HTTP
POST. No `initialize` handshake or session is needed for a single call:

```bash
curl -s https://www.maisoneclaire.co/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"search_products","arguments":{"query":"scented candle","lang":"en","limit":5}}}'
```

Use `"method":"tools/list"` to see the live tool schemas.

## Tools

All five tools are read-only (`readOnlyHint: true`, `destructiveHint: false`).

### search_products

Search the catalogue by keyword.

| Argument | Type | Notes |
| --- | --- | --- |
| `query` | string, required | 1–100 characters, any language ("bougie parfumée", "wax", "شمعة") |
| `category` | string | One of the category ids below |
| `lang` | `fr` \| `en` \| `ar` | Default `fr` |
| `in_stock_only` | boolean | Default `false` |
| `max_price` | number (MAD) | 1–100000. Also hides sold-out items |
| `limit` | integer | 1–20, default 10 |

Category ids:
- Candles and gifts: `bougies`, `packs`, `flower-box`, `decoration-de-table`, `cadeaux`,
  `cadeaux-mariage`, `cadeaux-pour-elle`, `cadeaux-ramadan`
- Supplies: `fournitures`, `cire-bougie` (wax), `meches` (wicks), `huiles-aromatiques`
  (fragrance oils), `colorants` (dyes), `moules` (moulds), `bougeoirs` (candle holders),
  `boule-de-neige` (snow globes), `kits`

Each result has `slug`, `name`, `short_description`, `category`, `in_stock`, `price_mad`,
`price_is_from`, `old_price_mad`, `image_url` and `url`. The response also has `total` and a
`search_url` for the same search on the website.

Example: "scented candles under 200 MAD, in English"

```json
{"query": "scented candle", "lang": "en", "max_price": 200, "in_stock_only": true}
```

Example: "soy wax for candle making"

```json
{"query": "soja", "category": "cire-bougie", "lang": "en"}
```

### get_product

Full details of one product: description, image gallery and variants (each with `label` and
`price_mad`).

| Argument | Type | Notes |
| --- | --- | --- |
| `slug` | string, required | From a search result, e.g. `pack-eclat-d-amour`. Lowercase letters, digits and hyphens |
| `lang` | `fr` \| `en` \| `ar` | Default `fr` |

```json
{"slug": "pack-eclat-d-amour", "lang": "en"}
```

An unknown slug returns an error ("Product not found"). Search again rather than guessing.

### list_categories

Every category with its id, label, page URL, group (`candles_gifts` or `supplies`) and product
count, plus links to the gift card and the build-your-own-box page. Use it to browse or to pick
a `category` id for `search_products`.

```json
{"lang": "en"}
```

### get_gift_ideas

In-stock gift suggestions for a budget and an occasion.

| Argument | Type | Notes |
| --- | --- | --- |
| `budget` | number (MAD), required | 30–100000 |
| `occasion` | string | `any` (default), `valentine`, `birthday`, `thank_you`, `for_her`, `wedding`, `wedding_guests`, `ramadan`, `eid`, `christmas` |
| `lang` | `fr` \| `en` \| `ar` | Default `fr` |
| `limit` | integer | 1–12, default 6 |

Example: "a birthday gift for my sister, about 300 dirhams"

```json
{"budget": 300, "occasion": "birthday", "lang": "en", "limit": 4}
```

### get_shop_info

The shop's current delivery, payment and return terms, plus links to the delivery, terms of sale,
FAQ, contact, gift card and gift guide pages. Call it whenever the user asks about delivery fees,
delivery time, minimum order, payment, returns or refunds.

```json
{"lang": "en"}
```

At the time of writing it says (check the live answer before you quote it):
- Delivery anywhere in Morocco in 2–3 days. The fee depends on the city zone (25 to 45 MAD), and
  the exact fee is shown at checkout. Delivery is free from 450 MAD of items.
- Minimum order: 100 MAD of items, excluding delivery.
- Cash on delivery only. You order on the website without an account, the shop calls to confirm,
  the courier calls before coming, and you check the parcel before paying.
- Returns within 7 days for unused items in their original packaging (return costs paid by the
  buyer), with the refund within 15 days. Personalised items and candles that have been lit can't
  be returned. A damaged or wrong item is replaced or refunded at no cost if reported with photos
  within 48 hours.

Point the user to the contact page for questions; don't share other contact details.

## Example flow

User: "I need a Ramadan gift box under 250 MAD delivered to Rabat."

1. `get_gift_ideas` with `{"budget": 250, "occasion": "ramadan", "lang": "en"}`.
2. Optionally `get_product` on the best match to read its contents.
3. `get_shop_info` with `{"lang": "en"}` for the Rabat delivery fee and the free-delivery
   threshold.
4. Reply with 2–3 options. Give each one's name, its price in MAD and its product URL. Then give
   the delivery fee for Rabat's zone, and say that payment is cash on delivery and that the order
   is placed on the website.

## Links

- Shop: https://www.maisoneclaire.co (English: https://www.maisoneclaire.co/en)
- Delivery: https://www.maisoneclaire.co/en/livraison
- Terms of sale: https://www.maisoneclaire.co/en/conditions-generales-de-vente
- FAQ: https://www.maisoneclaire.co/en/faq
- Contact: https://www.maisoneclaire.co/en/contactez-nous
- MCP server card: https://www.maisoneclaire.co/mcp/server-card
