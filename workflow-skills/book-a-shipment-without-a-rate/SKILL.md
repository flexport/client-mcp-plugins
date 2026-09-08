---
name: book-a-shipment-without-a-rate
description: "Create and submit an unrated supplier booking for consignee acceptance, when the user is acting as the shipper/supplier and doesn't have (or want) a priced rate. Flexport prices it for the consignee after submission. Two-phase preflight-then-confirm process — never skip the preflight, and never resubmit with a stale confirmation token after changing any booking detail."
disable-model-invocation: false
---

# book-a-shipment-without-a-rate

Calls `rates_book_without_rate`. Shipper/supplier role only — if the user's role is unclear, ask whether they'd rather get a rate first (route them to `get-instant-price-quote` / `request-a-custom-rate` instead). Dangerous goods and document uploads are not supported by this tool at all — if any DG screening question is true, stop and direct the user to the Flexport UI.

## Core rule: never silently reinterpret user input

Treat every value and every occurrence the user gives you as intentional. Never deduplicate, merge, omit, replace, or invent data to make it fit the schema. If something is ambiguous, contradictory, or doesn't map one-to-one (e.g. two HS code lines with the same code and description), stop and ask whether they're duplicates or genuinely separate lines — this tool rejects exact duplicate pairs, so guessing wrong fails the submission. Before submission, explicitly disclose anything you normalized, defaulted, or excluded.

## Step 1 — Collect and resolve, don't guess IDs [Research]

Summarize all required and conditionally-required fields up front before asking one-by-one.

- `consignee_entity_id` / `shipper_entity_id` — resolve via `network_search_company_entities`, picking an **active** entity with the `CONSIGNEE` or `SHIPPER` role respectively. Use its `company_entity_id`. Never guess this.
- `origin_address_fid` — resolve via `network_search_addresses` only. Google-address results are **not supported** here (unlike the instant-price flow). If nothing matches, tell the user to add the address in Flexport before retrying — don't fall back to a Google address.
- HS codes — if the shipper doesn't know a code, use `network_search_hs_codes`, present candidates, and ask them to pick one.
- If `wants_create_fulfillment_inbound` is true, resolve the destination via `network_search_fulfillment_inbound_addresses` using this booking's `origin_address_fid`, then confirm the choice with the user — this also limits the destination to Flexport warehouses.

## Step 2 — Dangerous goods, asked once [Confirm]

Ask a single combined question covering hazmat, lithium-ion/other batteries, magnets, and other dangerous goods — don't ask category by category. This tool only accepts `false` for every DG field. If the shipper confirms any DG present, stop here and send them to the Flexport UI instead of continuing.

## Step 3 — Service selection [Confirm]

Defaults depend on mode and incoterm — confirm the resolved choice with the user rather than silently applying it:
- Truck is always door-to-door pickup and delivery.
- Air/ocean: pickup defaults true for EXW and FCA Factory, false otherwise; delivery defaults false.
- Export customs defaults true only for EXW.
- If origin or the loading port is in Hong Kong, ask whether Flexport should handle Hong Kong trade declaration, and whether the cargo involves strategic/export-controlled goods (`declared_as_strategy`; provide `eccn_codes` if yes).

## Step 4 — Preflight (no token yet) [Write, unconfirmed]

Call `rates_book_without_rate` **without** `booking_confirmation_token`. This runs validated preflight only — it does not submit. Show the complete validated booking, including the fingerprint token it returns, to the shipper.

If the response includes non-blocking `policy_violations`: show them to the shipper and get their explicit review/confirmation before proceeding.

## Step 5 — Confirm and resubmit unchanged [Confirm → Write]

After the shipper explicitly confirms the preflighted booking (and any policy violations, if present — set `policy_violations_confirmed=true` on retry): call the tool again with **the exact same booking details** and the **unchanged** `booking_confirmation_token`.

**Any change to booking details — even one field — invalidates the token.** If the shipper changes anything after seeing the preflight, drop the token and go back to Step 4 for a fresh preflight. Don't try to reuse an old token with new details.

## Step 6 — Handle the result

Success: share `flex_id`, `url`, and the `next_action` text (e.g. `submitted`, `pending_consignee_quote_acceptance`) verbatim — it tells the user what happens next. Failure: `booking_id`/`flex_id`/`shipment_id` are null — read back whatever error context is available and don't retry blind.

## Trigger phrases

"book this without a rate", "submit this to the consignee for pricing", "I'm the supplier, book this shipment".
