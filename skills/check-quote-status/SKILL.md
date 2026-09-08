---
name: check-quote-status
description: "Look up the status of a previously submitted custom rate/quote request, and drill into its priced options. Use when the user asks 'what's the status of my quote request', 'has Flexport priced my rate request yet', 'show me the quote for FLEX-...', or wants to browse open/past quote requests by submitter, mode, or route."
disable-model-invocation: false
---

# check-quote-status

For quotes that came out of the `request-a-custom-rate` workflow (not instant-price search results). Three tools, used top-down: browse/find the request, then drill into its options, then a single option's full detail.

## Find the request

`rates_browse_quote_requests`:

- By name or FLEX-ID → pass `query`.
- By filters instead → `status` (defaults to `ACTIVE`, matching the web UI — use `ALL` only if the user explicitly wants historical requests), `modes` (`OCEAN`/`AIR`/`TRUCK`), location (resolve names first via `network_search_ports`/`network_search_addresses`, then pass `port_ids`/`address_fids` — matches either origin or destination).
- By submitter → don't pass a raw name into `requestor_ids`. Call `list_active_company_users` first (search by the name given, or list all if the user wants to pick), confirm the match with the user, then pass the resolved `user_id`.
- Paginate with `after`/`end_cursor` like other list tools here.

## Drill into one request

`rates_get_quote_request_details` — pass the `shipment_id` from the browse result. Returns the submission details plus **every** priced quote option for that request, with carrier, transit estimates, and cost per option.

## One specific priced option

`rates_get_quote_details` — pass a `quote_id` from either the above or a prior result. Returns full transit/route/rate-group detail, expiration, and the web action link for that single option.

When both `subtotal_display` and `subtotal` are present on a rate group, show `subtotal_display` — it's the intended display override.
