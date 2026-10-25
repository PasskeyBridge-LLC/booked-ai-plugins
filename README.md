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
- `bookings:write` is granted when the client requests it. The consent page shows it as a list item with no checkbox: "This assistant can book a visit and charge your saved card up to the shop's cap. If your bank asks you to confirm, or the card is declined, you pay on a Booked page. This stays on for this connection." To refuse it, decline the connection.
- `customer:self` is a list item with no checkbox: "This assistant can list, move and cancel your own bookings. It cannot cancel a booking that holds a payment."
- Member read scopes (`bookings:read`, `customers:read`, `staff:read`, `hours:read`, `settings:read`, `reviews:read`, `payments:read`, `team:read`) are shown for a shop owner, and for a signed-in member: "For shops you own or work at, this assistant can see bookings, customers (without names or contact details), staff, hours, settings, reviews, payments and the team, as far as your role allows."
- `shops:write` is an optional checkbox, shown to a shop owner: "Also let this assistant change the agent booking switch on shops you own. Leave this unchecked for a read-only connection." `get_shop_agent_config` and `set_shop_agent_config` need it.
- Member write scopes (`bookings:manage`, `customers:write`, `services:write`, `staff:write`, `hours:write`, `settings:write`, `reviews:write`) are one optional checkbox: "Also let this assistant make changes at shops you own or work at, as far as your role allows: bookings, customer tags, services, staff, hours, settings and review replies. It cannot take or refund payments, delete services, staff, customers or reviews, or change who has access."
- `offline_access` is granted when the client requests it.

A call is refused when the connection does not hold the tool's scope. Member and customer tools also check the account's role on the shop on every call. Floors: staff includes manager and owner; manager includes owner.

Authentication is OAuth with dynamic client registration. No API key or client secret is needed.

Setup guide: https://booked.co/help/shop-owners/setup/connect-claude

## Tools

`create_booking`, `book_returning`, and `confirm_deposit` are registered only when `MCP_EXPOSE_PAYMENT_TOOLS=1`.

Public:

| Tool | Parameters | What it returns | Scope |
|---|---|---|---|
| `get_shop` | slug | Public shop for that slug: services (name, duration, price), staff, hours, booking rules, payment mode, deposit percent, tipping. | `shops:read` |
| `list_services` | | The shop's services, with duration and price. | `shops:read` |
| `get_availability` | `shop_id`, `date`; `service_id` optional | Schedules, time off, and bookings for that date. With `service_id`, includes that service's `service_capacity`. | `availability:read` |
| `whoami` | | The signed-in principal. | Any signed-in connection |
| `get_app_link` | | A Booked app link. Reads no data. | `shops:read` |

Booking. `bookings:write`. A principal with no trust score cannot open a deposit. The shop's agent switch and daily cap apply.

| Tool | Who | What it does |
|---|---|---|
| `create_booking` | Any connection with the scope | Starts a deposit booking. The shop sets the amount. |
| `book_returning` | The signed-in customer | Books from that customer's past visits. A saved card is charged only up to the shop's cap. If the bank asks the customer to confirm, or the card is declined, the customer pays on a Booked page. Saved-card charges stay off until the shop turns them on and sets a maximum. No maximum refuses the charge. |
| `confirm_deposit` | Any connection with the scope | Checks whether the deposit payment has completed. |

The assistant cannot turn saved-card charges on or set that maximum. `set_shop_agent_config` refuses `agent_off_session_enabled` and `agent_booking_cap_cents` and writes nothing when either is sent.

Agent config. `shops:write` on a shop the owner consents for.

| Tool | Parameters | What it does |
|---|---|---|
| `get_shop_agent_config` | `shop_id` | `agent_bookings_enabled`, `daily_agent_booking_limit`, bookings taken today, and the saved-card settings. |
| `set_shop_agent_config` | `shop_id`, and at least one of `agent_bookings_enabled` (boolean) or `daily_agent_booking_limit` (integer, or null to remove the cap) | Turns agent bookings on or off, and sets or removes the daily cap. An omitted field stays as stored. |

Customer. `customer:self`.

| Tool | Parameters | What it does |
|---|---|---|
| `list_my_bookings` | `upcoming_only`, `limit`, `cursor` | The account's bookings: shop, service, staff display name, time, price, status. `upcoming_only` keeps pending and confirmed visits that have not ended. |
| `cancel_my_booking` | `booking_id` | Cancels that booking. Refuses `money_held` when a payment is held. An already canceled booking returns `already_canceled`. |
| `reschedule_my_booking` | `booking_id`, `new_starts_at` | Moves that booking. Refuses `deposit_at_risk` when a captured deposit is past the refund window. |

Shop members. `limit` and `cursor` page the lists that take them. Customer rows do not include the customer's name, email, or phone.

| Tool | Parameters | Floor | Scope |
|---|---|---|---|
| `list_bookings` | `shop_id`, `from`, `to`, `staff_id`, `status`, `limit`, `cursor` | staff | `bookings:read` |
| `get_booking` | `booking_id` | staff | `bookings:read` |
| `list_waitlist` | `shop_id`, `date`, `limit`, `cursor` | staff | `bookings:read` |
| `list_customers` | `shop_id`, `tag`, `limit`, `cursor` | staff | `customers:read` |
| `get_customer_history` | `customer_id`, `limit`, `cursor` | staff | `customers:read` |
| `list_staff` | `shop_id` | staff | `staff:read` |
| `get_hours` | `shop_id`, `from`, `to` | staff | `hours:read` |
| `get_shop_settings` | `shop_id` | manager | `settings:read` |
| `list_locations` | `shop_id` | manager | `settings:read` |
| `get_action_limits` | `shop_id` | manager | `settings:read` |
| `list_mcp_audit` | `shop_id`, `from`, `to`, `limit`, `cursor` | owner | `settings:read` |
| `create_member_booking` | `shop_id`, `service_id`, `staff_id`, `starts_at`, `customer_id` | manager | `bookings:manage` |
| `create_booking_series` | `shop_id`, `service_id`, `staff_id`, `first_starts_at`, `interval_weeks`, `occurrence_count`, `customer_id` | manager | `bookings:manage` |
| `reschedule_booking` | `booking_id`, `new_starts_at`, `duration_minutes` | manager | `bookings:manage` |
| `cancel_booking_series` | `series_id` | manager | `bookings:manage` |
| `cancel_booking` | `booking_id`, `notify` | staff | `bookings:manage` |
| `check_in_booking` | `booking_id`, `checked_in` | staff | `bookings:manage` |
| `set_booking_status` | `booking_id`, `status` | staff | `bookings:manage` |
| `send_booking_notice` | `booking_id`, `kind` | staff | `bookings:manage` |
| `tag_customer` | `customer_id`, `tag` | staff | `customers:write` |
| `untag_customer` | `customer_id`, `tag` | staff | `customers:write` |
| `create_service` | `shop_id`, `name`, `description`, `duration_minutes`, `price_cents`, `capacity`, `is_active` | manager | `services:write` |
| `update_service` | `service_id`, `name`, `description`, `clear_description`, `duration_minutes`, `price_cents`, `capacity`, `is_active` | manager | `services:write` |
| `archive_service` | `service_id`, `archived` | manager | `services:write` |
| `copy_location_services` | `from_shop_id`, `to_shop_id` | owner | `services:write` |
| `create_staff` | `shop_id`, `display_name`, `color`, `bio`, `notify_on_booking`, `is_active`, `service_ids`, `overrides` | manager | `staff:write` |
| `update_staff` | `staff_id`, `display_name`, `color`, `bio`, `clear_bio`, `notify_on_booking`, `is_active`, `service_ids`, `overrides` | manager | `staff:write` |
| `deactivate_staff` | `staff_id`, `active` | manager | `staff:write` |
| `set_weekly_hours` | `shop_id`, `hours` | manager | `hours:write` |
| `add_hours_override` | `shop_id`, `on_date`, `ranges` | manager | `hours:write` |
| `remove_hours_override` | `override_id` | manager | `hours:write` |
| `set_staff_schedule` | `staff_id`, `week` | manager | `hours:write` |
| `add_time_off` | `shop_id`, `staff_id`, `starts_at`, `ends_at` | manager | `hours:write` |
| `remove_time_off` | `block_id` | manager | `hours:write` |
| `update_shop_settings` | `shop_id`, `settings` | manager | `settings:write` |
| `lower_action_limits` | `shop_id`, `action_class`, `disable`, `per_call_cents`, `daily_cents`, `daily_count` | manager | `settings:write` |
| `set_deposit_policy` | `shop_id`, `payment_mode`, `deposit_percent` | owner | `settings:write` |
| `set_no_show_policy` | `shop_id`, `policy`, `fee_cents` | owner | `settings:write` |
| `set_tipping` | `shop_id`, `enabled`, `preset_pct`, `allow_custom` | owner | `settings:write` |
| `set_gift_cards_enabled` | `shop_id`, `enabled` | owner | `settings:write` |
| `upsert_pass_offer` | `shop_id`, `offer_id`, `kind`, `name`, `price_cents`, `visits`, `service_id`, `active` | owner | `settings:write` |
| `set_pass_offer_active` | `offer_id`, `active` | owner | `settings:write` |
| `list_reviews` | `shop_id`, `hidden`, `limit`, `cursor` | staff | `reviews:read` |
| `reply_to_review` | `review_id`, `reply` | manager | `reviews:write` |
| `hide_review` | `review_id` | manager | `reviews:write` |
| `unhide_review` | `review_id` | manager | `reviews:write` |
| `list_gift_cards` | `shop_id`, `limit`, `cursor` | manager | `payments:read` |
| `list_pass_offers` | `shop_id` | manager | `payments:read` |
| `list_customer_passes` | `shop_id`, `customer_id`, `limit`, `cursor` | manager | `payments:read` |
| `list_payments` | `shop_id`, `from`, `to`, `limit`, `cursor` | manager | `payments:read` |
| `get_billing` | `shop_id` | owner | `payments:read` |
| `list_members` | `shop_id` | owner | `team:read` |

`list_bookings` requires `from` and `to`, at most 93 days apart. `status` is `pending`, `confirmed`, `canceled`, `completed`, or `no_show`. Owner and manager rows include `deposit_state`. Booking notes are returned as `has_notes`, not the text. `list_customers` returns `has_email`, `has_phone`, `has_notes`, and `has_saved_card`, and drops a tag that contains `@` or a run of seven digits.

Refunds, other charges, deletions, role changes, and security settings are declared and not registered.

## Example prompts

- What services does this shop on Booked offer, and what do they cost?
- What is the availability for a haircut next Tuesday?
- Book the 3 PM slot.
- Which bookings does this shop have tomorrow? (staff, manager, or owner)
- Move Thursday's 2 PM appointment to Friday at the same time. (manager or owner)

## Payments

A deposit is paid on a Booked page. When the shop has turned saved-card charges on and set a maximum, a booking can charge the saved card up to that maximum. The server does not take card numbers. No registered tool refunds a payment.

## Privacy and terms

- Privacy policy: https://booked.co/privacy
- Terms of service: https://booked.co/terms

## Support

hello@booked.co

## License

MIT. See [LICENSE](LICENSE).
