# Booked plugin for Cursor

Connects Cursor to the Booked remote MCP server at `https://booked.co/api/mcp`. Look up a shop on Booked, list its services and open times, and book an appointment. Booked is scheduling software for salons and barbers: https://booked.co

## Install

From the Cursor Marketplace: open the Booked plugin page in the [Cursor Marketplace](https://cursor.com/marketplace) and install it.

With an install link (opens Cursor and adds the server):

```text
cursor://anysphere.cursor-deeplink/mcp/install?name=booked&config=eyJ1cmwiOiJodHRwczovL2Jvb2tlZC5jby9hcGkvbWNwIn0=
```

By hand, add this to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{
  "mcpServers": {
    "booked": {
      "url": "https://booked.co/api/mcp"
    }
  }
}
```

## Connect

1. Cursor opens the Booked sign-in page in your browser. Sign in.
2. The consent page shows what the connection may do. `bookings:write` is always granted. `shops:write` is optional and is needed only for `set_shop_agent_config`. Approve.
3. The browser returns to Cursor and the Booked tools are available.

Authentication is OAuth with dynamic client registration. No API key or client secret is needed.

Setup guide: https://booked.co/help/shop-owners/setup/connect-claude

## Tools

Read-only:

| Tool | What it does |
|---|---|
| `get_shop` | Returns a shop's public details. |
| `list_services` | Lists the services a shop offers. |
| `get_availability` | Returns open appointment times for a service. |
| `whoami` | Returns the account the connection is signed in as. |
| `get_shop_agent_config` | Returns the shop's settings for AI agent bookings. |

Write:

| Tool | What it does |
|---|---|
| `create_booking` | Creates an appointment booking. |
| `confirm_deposit` | Checks whether the deposit for a booking has been paid. |
| `set_shop_agent_config` | Updates the shop's settings for AI agent bookings. Requires `shops:write`. |

## Example prompts

- What services does this shop on Booked offer, and what do they cost?
- Which times are open for a haircut next Tuesday afternoon?
- Book the 3 PM slot and send me the deposit link.

## Payments

When a booking needs a deposit, the customer pays it on a Booked-hosted Stripe Checkout page. The MCP server moves no money and handles no card data.

## Privacy and terms

- Privacy policy: https://booked.co/privacy
- Terms of service: https://booked.co/terms

## Support

hello@booked.co

## License

MIT. See [LICENSE](LICENSE).
