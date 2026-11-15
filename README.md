# CouponSwift MCP server: deals and promo codes for software, hosting, VPN, proxy and AI tools

A hosted (remote) [Model Context Protocol](https://modelcontextprotocol.io) server for CouponSwift (https://www.couponswift.com), a site that lists deals and promo codes for software, hosting, VPN, proxy and AI tools. It has one read-only tool that finds a store by name, slug or domain and returns a link to its CouponSwift page together with a short summary of the current offers. The codes themselves are shown on the store page on the website, not in the tool response.

This repository contains documentation only. There is nothing to install or run.

## Endpoint

```
https://www.couponswift.com/mcp
```

- Transport: Streamable HTTP (JSON-RPC over `POST`). `GET` returns `405`.
- Authentication: none.
- Stateless: no session is required.
- Protocol revisions accepted: `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`.

Use the URL exactly as written above, with `www`.

## Tool: `find_store`

Finds stores on CouponSwift by brand name, store slug or website domain (for example `hostinger`, `bunny.net` or `https://www.nordvpn.com/pricing`). Returns up to 5 matching stores, best match first.

For each store it returns:

| Field | Type | Meaning |
| --- | --- | --- |
| `name` | string | Store name. |
| `category` | string or null | Top-level category name. |
| `maxDiscount` | integer or null | Highest percentage discount among the active offers; null when no offer states a percentage. |
| `activeOffers` | integer | Number of active offers shown on the store page. |
| `url` | string | The store page on CouponSwift, `https://www.couponswift.com/{slug}/coupons`. |

Annotations: `readOnlyHint: true`, `openWorldHint: false`.

### Input schema

```json
{
  "type": "object",
  "properties": {
    "query": {
      "type": "string",
      "minLength": 2,
      "maxLength": 80,
      "description": "A brand name, a store slug or a website domain or URL, 2 to 80 characters."
    }
  },
  "required": ["query"],
  "additionalProperties": false
}
```

### Output schema

```json
{
  "type": "object",
  "properties": {
    "stores": {
      "type": "array",
      "maxItems": 5,
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string", "description": "Store name." },
          "category": { "type": ["string", "null"], "description": "Top-level category name, or null." },
          "maxDiscount": { "type": ["integer", "null"], "description": "Highest percentage discount among the active offers, or null." },
          "activeOffers": { "type": "integer", "description": "Number of active offers shown on the store page." },
          "url": { "type": "string", "description": "The store page on CouponSwift." }
        },
        "required": ["name", "category", "maxDiscount", "activeOffers", "url"],
        "additionalProperties": false
      }
    }
  },
  "required": ["stores"],
  "additionalProperties": false
}
```

### Example

Request:

```bash
curl -s https://www.couponswift.com/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"find_store","arguments":{"query":"hostinger"}}}'
```

Example response (the numbers change as offers change):

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "isError": false,
    "content": [
      {
        "type": "text",
        "text": "Hostinger: up to 85% off, 29 active offers. https://www.couponswift.com/hostinger/coupons"
      }
    ],
    "structuredContent": {
      "stores": [
        {
          "name": "Hostinger",
          "category": "Web Hosting",
          "maxDiscount": 85,
          "activeOffers": 29,
          "url": "https://www.couponswift.com/hostinger/coupons"
        }
      ]
    }
  }
}
```

When nothing matches, `stores` is an empty array and the text reads `No store matched "<query>".`

## Add it to a client

### Claude Code

```bash
claude mcp add --transport http couponswift https://www.couponswift.com/mcp
```

### Claude (web and desktop apps)

Add it as a custom connector: open **Customize > Connectors**, click **+ Add**, then **Add custom connector**, enter a name and the URL `https://www.couponswift.com/mcp`. See [Getting started with custom connectors using remote MCP](https://support.claude.com/en/articles/11175166-getting-started-with-custom-connectors-using-remote-mcp). On Team and Enterprise plans an owner adds custom connectors.

### Cursor

In `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{
  "mcpServers": {
    "couponswift": {
      "url": "https://www.couponswift.com/mcp"
    }
  }
}
```

### VS Code (GitHub Copilot)

In `~/.copilot/mcp-config.json` (all workspaces) or `.mcp.json` at the workspace root:

```json
{
  "mcpServers": {
    "couponswift": {
      "type": "http",
      "url": "https://www.couponswift.com/mcp"
    }
  }
}
```

You can also run **Chat: Open Customizations**, select **MCP Servers**, then **Add Server** and enter the URL.

### Clients without remote server support

Use [`mcp-remote`](https://github.com/geelen/mcp-remote) as a local bridge (requires Node.js):

```json
{
  "mcpServers": {
    "couponswift": {
      "command": "npx",
      "args": ["mcp-remote", "https://www.couponswift.com/mcp"]
    }
  }
}
```

## What it does not do

- It does not return coupon codes. Codes are shown on the store page on the website.
- It does not return affiliate links or tracking URLs. The only link it returns is the CouponSwift store page.
- It does not expose offer details, expiry dates or history.
- It has no tools that change anything.

## Limits

- Read-only, one tool.
- No authentication and no user data. Queries are 2 to 80 characters.
- At most 5 stores per call.
- Stores that are not listed on CouponSwift are not returned.

## Links

- Website: https://www.couponswift.com
- Contact: https://www.couponswift.com/contact

## License

MIT, see [LICENSE](LICENSE). The license covers the files in this repository. The hosted service and the data it returns are not part of this repository.
