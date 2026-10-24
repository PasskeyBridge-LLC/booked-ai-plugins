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

1. Cursor opens the Booked sign-in page in your browser. Sign in with a shop owner account or a customer account. A customer account can approve `bookings:write`.
2. The consent page lists the scopes. Approve.
3. The browser returns to Cursor and the Booked tools are available.

Shop owners and customers can revoke a connection from their account.

Scopes:

- `shops:read` and `availability:read` are always granted.
- `bookings:write` is granted when the client requests it. The consent page shows it as a list item with no checkbox. To refuse it, decline the connection.
- `shops:write` is optional. It is granted only when the client requests it and you leave it checked, and it applies only to shops you own. `get_shop_agent_config` and `set_shop_agent_config` need it.
- Member read scopes (`bookings:read`, `customers:read`, `staff:read`, `hours:read`, `settings:read`, `reviews:read`, `payments:read`, `team:read`): for a shop owner, manager or staff account, the consent page says the assistant can see bookings, customers (without names or contact details), staff, hours, settings, reviews, payments and the team at shops you own or work at, as far as your role allows.
- Member write scopes (`bookings:manage`, `customers:write`, `services:write`, `staff:write`, `hours:write`, `settings:write`, `reviews:write`) are optional: a checkbox you leave checked to let the assistant make changes, as far as your role allows. It cannot take or refund payments, delete services, staff, customers or reviews, or change who has access.
- `customer:self` lets a customer account see, cancel and reschedule its own bookings.
- `offline_access` is granted when the client requests it.

Every member and customer tool checks your role in the database on every call. A tool is listed only when the connection holds its scope.

Authentication is OAuth with dynamic client registration. No API key or client secret is needed.

Setup guide: https://booked.co/help/shop-owners/setup/connect-claude

## Tools

Public and booking:

| Tool | What it does | Access |
|---|---|---|
| `get_shop` | Retrieve public details for a shop by slug: services, staff, policies and payment settings. | `shops:read` |
| `list_services` | List the services a shop offers, with price and duration. | `shops:read` |
| `get_availability` | Open times for a shop on one date. When service_id is set, the result includes the seats left for that group class. | `availability:read` |
| `whoami` | Return the authenticated principal, its scopes, trust score and the shops it may manage. | Any signed-in connection |
| `get_app_link` | The Booked app page for something MCP does not carry: contact details, a booking's notes and payments, the billing card, bank details, locations, account settings. Reads no data. | `shops:read` |
| `get_shop_agent_config` / `set_shop_agent_config` | Read, or turn on or off, agent bookings for a shop and its daily cap. | Admin, or `shops:write` on a shop this connection manages |
| `create_booking` | Start a booking that needs the shop's deposit. Returns a hosted payment URL; the booking exists only after the customer pays. Subject to the shop's agent switch, daily cap and the trust score. | `bookings:write` |
| `book_returning` | Book the signed-in customer's usual shop, service, staff member and time, from their own past visits. Charges the saved card when the shop allows it, otherwise returns a payment URL. | `bookings:write`, the customer |
| `confirm_deposit` | Check whether the hosted deposit has been paid and the booking exists. | `bookings:write` |

`create_booking`, `book_returning` and `confirm_deposit` are listed only while Booked has its payment tools switched on.

For shop owners, managers and staff (role shown in brackets):

| Area | Read | Change |
|---|---|---|
| Bookings | `list_bookings`, `get_booking`, `list_waitlist` | `create_member_booking`, `create_booking_series`, `reschedule_booking`, `cancel_booking_series` (owner, manager); `cancel_booking`, `check_in_booking`, `set_booking_status`, `send_booking_notice` (any member) |
| Customers | `list_customers`, `get_customer_history` | `tag_customer`, `untag_customer` |
| Services | | `create_service`, `update_service`, `archive_service` (owner, manager); `copy_location_services` (owner) |
| Staff | `list_staff` | `create_staff`, `update_staff`, `deactivate_staff` (owner, manager) |
| Hours | `get_hours` | `set_weekly_hours`, `add_hours_override`, `remove_hours_override`, `set_staff_schedule`, `add_time_off`, `remove_time_off` (owner, manager) |
| Settings | `get_shop_settings`, `list_locations`, `get_action_limits` (owner, manager); `list_mcp_audit` (owner) | `update_shop_settings`, `lower_action_limits` (owner, manager); `set_deposit_policy`, `set_no_show_policy`, `set_tipping`, `set_gift_cards_enabled`, `upsert_pass_offer`, `set_pass_offer_active` (owner) |
| Reviews | `list_reviews` | `reply_to_review`, `hide_review`, `unhide_review` (owner, manager) |
| Payments | `list_gift_cards`, `list_pass_offers`, `list_customer_passes`, `list_payments` (owner, manager); `get_billing` (owner) | None. No tool takes, refunds or moves money. |
| Team | `list_members` (owner) | None |

For a customer account: `list_my_bookings`, `cancel_my_booking`, `reschedule_my_booking` (`customer:self`).

Customer names and contact details are never returned. Refunds, charges, deletions, role changes and security settings are not available through MCP; do them in the Booked app.

## Example prompts

- What services does this shop on Booked offer, and what do they cost?
- Which times are open for a haircut next Tuesday afternoon?
- Book the 3 PM slot and give me the deposit link.
- Which bookings do I have tomorrow, and who is working? (shop owner, manager or staff)
- Move Thursday's 2 PM appointment to Friday at the same time. (owner or manager)

## Payments

When a booking needs a deposit, the customer pays it on a Booked-hosted Stripe Checkout page. The MCP server moves no money and handles no card data.

## Privacy and terms

- Privacy policy: https://booked.co/privacy
- Terms of service: https://booked.co/terms

## Support

hello@booked.co

## License

MIT. See [LICENSE](LICENSE).
