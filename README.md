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

1. Cursor opens the Booked sign-in page in your browser. Sign in with an account that owns at least one shop. Accounts that do not own a shop cannot approve a connection.
2. The consent page lists the scopes. Approve.
3. The browser returns to Cursor and the Booked tools are available.

Scopes:

- `shops:read` and `availability:read` are always granted.
- `bookings:write` is granted when the client requests it. The consent page shows it checked and you cannot uncheck it; to refuse it, decline the connection.
- `shops:write` is optional. It is granted only when the client requests it and you leave it checked, and it applies only to shops you own. `get_shop_agent_config` and `set_shop_agent_config` need it.
- `offline_access` is granted when the client requests it.

Authentication is OAuth with dynamic client registration. No API key or client secret is needed.

Setup guide: https://booked.co/help/shop-owners/setup/connect-claude

## Tools

Read-only:

| Tool | What it does | Access |
|---|---|---|
| `get_shop` | Retrieve public details for a shop by slug: services, staff, policies and payment settings. | `shops:read` |
| `list_services` | List the services a shop offers, with price and duration. | `shops:read` |
| `get_availability` | Open times for a shop on one date. When service_id is set, the result includes the seats left for that group class. | `availability:read` |
| `whoami` | Return the authenticated principal, its scopes, trust score and the shops it may manage. | Any signed-in connection |
| `get_shop_agent_config` | Return whether agent bookings are on for a shop, and its daily cap. | Admin, or `shops:write` on a shop this connection manages |

Write:

| Tool | What it does | Access |
|---|---|---|
| `create_booking` | Start a booking that needs the shop's deposit. Returns a hosted payment URL. The customer pays on that page. The booking exists only after the deposit is paid. The amount comes from the shop. Free or unpaid bookings are not allowed. Subject to the shop's agent switch and daily cap, and to the trust score. | `bookings:write` |
| `confirm_deposit` | Check whether the hosted deposit has been paid and the booking exists. Pass the checkout_session_id from create_booking. Returns the booking reference once Stripe has the payment. | `bookings:write` |
| `set_shop_agent_config` | Turn agent bookings on or off for a shop, and set or remove its daily cap. At least one field is required. | Admin, or `shops:write` on a shop this connection manages |

## Example prompts

- What services does this shop on Booked offer, and what do they cost?
- Which times are open for a haircut next Tuesday afternoon?
- Book the 3 PM slot and give me the deposit link.

## Payments

When a booking needs a deposit, the customer pays it on a Booked-hosted Stripe Checkout page. The MCP server moves no money and handles no card data.

## Privacy and terms

- Privacy policy: https://booked.co/privacy
- Terms of service: https://booked.co/terms

## Support

hello@booked.co

## License

MIT. See [LICENSE](LICENSE).
