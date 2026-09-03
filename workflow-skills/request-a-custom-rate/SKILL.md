---
name: request-a-custom-rate
description: "Request a new custom rate quote from Flexport's account team for a lane, when instant pricing doesn't have an acceptable option. Use when the user wants a fixed/contracted rate, instant search came back empty, or they explicitly ask to 'request a quote' or 'get a custom rate'. Requires explicit user confirmation before submitting — never call the underlying tool without it."
disable-model-invocation: false
---

# request-a-custom-rate

Calls `rates_request_rate`. This creates a real request to Flexport's account team — every step below is a hard precondition the tool itself enforces; skipping one is not just bad practice, it's the wrong call.

## Step 1 — Check instant pricing first [Research]

You MUST call `rates_search_instant_price` before this tool, even if the user already said "I want a custom quote." Use the `get-instant-price-quote` skill for this.

- **No rates at all** → proceed to Step 2 with `offering_contract_type=HEDGE_FAK` (spot/FAK) as the default, unless the user wants a fixed rate.
- **Only spot rates exist, but the user wants a fixed rate** → proceed with `offering_contract_type=LONG_NAC` (Named Account Contract).
- **Acceptable instant rates exist** → stop. Point the user to booking instead (the `book-an-instant-rate` workflow-skill). Do not request a custom rate just because one is possible.

## Step 2 — Confirm the path when there are truly no rates [Confirm]

If instant search found nothing at all, don't jump straight to this tool. Ask the user to choose between:
- Booking without a rate (`book-a-shipment-without-a-rate` workflow-skill — shipper/supplier only), or
- Requesting a new rate (continue below).

Only continue with this skill if they pick the latter.

## Step 3 — Resolve IDs, never trust user input directly [Research]

- Origin/destination port → `network_search_ports`, use `port_id`.
- Origin/destination address → `network_search_addresses`, use `address_fid`. If nothing matches, fall back to `network_search_google_addresses` and use its `commerce_address_fid`.

Never pass a port ID or address FID straight from the user — always confirm it resolves to a real match first.

## Step 4 — Collect the rest and confirm before submitting [Confirm]

Required: `transportation_mode` (air / ocean_fcl / ocean_lcl), `incoterm`, `cargo_ready_date`. Optional but valuable: `freight_type` (port_to_port / port_to_door / door_to_port / door_to_door), `offering_contract_type`, and a `note` covering cargo profile, target price/budget, and expected volume — collect these proactively, they speed up the account team's turnaround.

**Present the complete request back to the user and get explicit confirmation to submit.** Do not call `rates_request_rate` without it — there is no preflight/undo step for this tool the way there is for bookings.

## Step 5 — Submit

Call `rates_request_rate`. On success, tell the user the `client_request_id` and the `nextAction` text verbatim — it describes what happens next (HEDGE_FAK ≈ 2 business days; LONG_NAC timeline communicated by the account team). On failure, surface the `errors` array and stop — don't retry silently.

## Trigger phrases

"request a custom rate", "get a quote for this lane", "ask Flexport for pricing", "no instant rate available, what now".
