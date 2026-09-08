---
name: get-instant-price-quote
description: "Search instant freight rates (Ocean FCL, Ocean LCL, Air) and get an itemized evaluated price for a specific option, without booking. Use when the user asks 'what would it cost to ship X from A to B', 'get me a quote', 'instant price', or wants to compare rate options before deciding whether to book. Not for dangerous goods or trucking-only (FTL) — those aren't supported by instant pricing."
disable-model-invocation: false
---

# get-instant-price-quote

Two tools, always used in sequence: `rates_search_instant_price` to find options, then `rates_evaluate_total_price_from_instant_price_search` to get a detailed price for the one the user picks. This skill stops at "here's the price" — booking is a separate step (see the `book-an-instant-rate` workflow-skill).

## Before searching

Ask anything not already provided — do not guess:

1. **Dangerous goods check first.** Instant pricing does not support DG/hazmat cargo. If the user confirms DG is present, decline instant pricing entirely (point them to a custom quote via `request-a-custom-rate` instead).
2. **Pickup/delivery scope** — ask two separate questions unless the user has already made it obvious (e.g. named a specific port for a side):
   - Should Flexport handle pickup from the origin, or does the shipment start at a port?
   - Should Flexport handle delivery to the destination, or does it end at a port?
   Combine the answers into `freight_type`: `DOOR_TO_DOOR`, `DOOR_TO_PORT`, `PORT_TO_DOOR`, or `PORT_TO_PORT`.
3. **Resolve each port/address and confirm it with the user before searching** — never pass a name straight through:
   - Port side → `network_search_ports`, use its `port_id`.
   - Address side, onboarded in Flexport → `network_search_addresses`, use `address_fid`.
   - Address side, not onboarded (city, postal code, street address) → `network_search_google_addresses`, use `commerce_address_fid`.
4. **Cargo details**: `transportation_mode` (OCEAN_FCL / OCEAN_LCL / AIR), `incoterm`, `cargo_ready_date`. For OCEAN_LCL/AIR, prefer per-package `package_details`; fall back to total weight (kg) + total volume (cbm) only if the user can't give per-package dimensions.

## Search

Call `rates_search_instant_price`. If `empty_reason` comes back set:
- `AIR_CARGO_OVERSIZED` — one or more packages exceed regional limits.
- `NO_RESULTS` — nothing available on this lane.

Either way: tell the user, then ask them to choose between `rates_book_without_rate` (the `book-a-shipment-without-a-rate` workflow-skill, shipper/supplier role only) or requesting a new rate (the `request-a-custom-rate` workflow-skill). Do not call either automatically.

## Evaluate the selected option

Once the user picks a result:

1. **Add-ons** — read `addon_service_options` on that result. Anything with `force_included=true` is mandatory (tell the user it's included, don't ask). For everything else, ask explicitly yes/no.
2. **Detention** — if `detention_addon_options` is non-empty, show the tiers (`total_free_days` + `addon_price`) next to the base `detention_base_free_days` and ask if they want to upgrade. If yes, the add-on value to pass later is `total_free_days − detention_base_free_days` (add-on days only, not the total).
3. Call `rates_evaluate_total_price_from_instant_price_search` with `search_snapshot_id` + `item_snapshot_id` from the search result, the `wants_*` flags set per the confirmed add-on choices (including the `force_included=true` ones), and `detention_addon_free_days` if upgraded.
4. Present `total_price` and the full `charge_line_items` breakdown to the user.

The response includes a `price_confirmation_token` — hand this, along with the exact same snapshot IDs and `wants_*`/detention flags, to the `book-an-instant-rate` workflow-skill if the user decides to book. It's a workflow-consistency fingerprint, not a security credential — it only proves the booking matches what was just evaluated, not who ran this tool.
