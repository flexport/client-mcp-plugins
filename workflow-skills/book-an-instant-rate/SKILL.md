---
name: book-an-instant-rate
description: "Create a real, binding booking from an instant-price rate the user has already reviewed and priced. Use only after the get-instant-price-quote skill has produced an evaluated price the user has explicitly accepted — never call this speculatively or to 'see what it would cost'. This tool creates a real booking that cannot be undone through this interface."
disable-model-invocation: false
---

# book-an-instant-rate

Calls `rates_instant_book`. **This creates a real, binding booking. Never call it speculatively, and never call it before the user has explicitly said go ahead on a specific, already-evaluated price.**

## Two modes — use snapshot mode unless the user hands you a rate ID directly

- **Snapshot mode (preferred, normal path):** `search_snapshot_id` + `item_snapshot_id`, following on from the `get-instant-price-quote` skill. Cargo details, cargo-ready date, route, and incoterm are all reused automatically from that search — do not resupply them, and don't ask the user for them again.
- **`client_rate_id` mode:** only if the user directly supplies a rate identifier themselves (no tool here returns one — never guess or fabricate it). In this mode, `cargo_ready_date` and `cargo_details` ARE required from you, since there's no snapshot to reuse them from.

Provide exactly one mode's fields. Never mix the two.

## Step 1 — Confirm you have a matching price confirmation (snapshot mode only)

`price_confirmation_token` is required in snapshot mode — it's the token `rates_evaluate_total_price_from_instant_price_search` returned in the `get-instant-price-quote` skill, for this *exact* `search_snapshot_id`, `item_snapshot_id`, `wants_*` flags, and `detention_addon_free_days`. If you don't have one yet, or anything about the selection changed since it was issued, go run `get-instant-price-quote` (or re-run the evaluate step) first — don't guess or reuse a token from a different selection. If the token is missing or stale, this tool returns `REQUIRES_PRICE_CONFIRMATION` rather than booking.

`client_rate_id` mode has no evaluate step and no token — but the "show the full booking and get explicit go-ahead" rule below still applies exactly the same.

## Step 2 — NAC allocation weeks, if present [Confirm]

If the search result included `nac_allocation` (a NAC rate with weekly allocation), you must, before calling this tool: present the feasible weeks to the user, showing each week's `week_date_range` and `available_allocation` next to the shipment's `requested_teus`, and ask them to pick one. Pass the chosen week/year as `ssat_allocation_preference`. **Never pick a week silently.**

## Step 3 — Final go-ahead [Confirm]

Show the user the evaluated price and every add-on/detention selection one more time, and get explicit confirmation to book — regardless of mode, regardless of whether you already confirmed it during evaluation. This is the last checkpoint before an irreversible action.

## Step 4 — Book

Call `rates_instant_book`. On failure, `success=false` and `error_code` will be one of `RATE_LOOKUP_FAILED`, `RATE_NOT_FOUND`, `RATE_CLIENT_MISMATCH`, `RATE_NOT_ACTIVE`, `RATE_EXPIRED`, `RATE_NOT_YET_EFFECTIVE`, `CARGO_MODE_MISMATCH`, `BOOKING_FAILED`, `BOOKING_INCOMPLETE`, `SNAPSHOT_NOT_FOUND`, `SNAPSHOT_CLIENT_MISMATCH`, `MODE_NOT_ENABLED`, or `REQUIRES_PRICE_CONFIRMATION` — read `error_message` to the user verbatim and stop; don't retry automatically. On success, share the `flex_id` and `url` so the user can see the created shipment.

## Trigger phrases

"book it", "go ahead and book this rate", "confirm the booking" — only after an evaluated price from `get-instant-price-quote` has been shown and accepted.
