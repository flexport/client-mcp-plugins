---
name: track-shipments
description: "Find and check on Flexport shipments by FLEX-ID, name, tag, or filters (status, mode, date range, demurrage/detention risk, open tasks). Use when the user asks 'where is my shipment', 'track FLEX-...', 'what's arriving this week', 'show me shipments at risk of demurrage', or wants a list/status of their freight."
disable-model-invocation: false
---

# track-shipments

Two tools cover shipment lookup — pick based on whether the user named a specific shipment or wants a filtered list.

## Which tool to use

- **`track_shipment`** — the user named a specific shipment: a FLEX-ID, a shipment name, or a client-assigned tag. Supply exactly one of `flex_id`, `name`, or `tag`. `flex_id` is exact-match only; `name` and `tag` fall back to fuzzy matches if there's no exact one. Returns up to 20 matches.
- **`browse_shipments`** — the user wants a filtered list or count, not one specific shipment: "shipments arriving this week", "anything with an open task", "ocean shipments at demurrage risk in the next 5 days". Paginated, up to 100 per page via `first`/`after`.

Both return the same tracking detail shape: route stops, containers with last-free-day info, customs entries with PGA hold status, exceptions, open work-item tasks, and metadata tags.

## Filter semantics that are easy to get wrong (`browse_shipments`)

- **`is_completed`**: not implied by status. A shipment can sit at `FINAL_DESTINATION` and still not be administratively complete — the two are tracked independently and combined with AND. Default to `false` for a general "what's active" request; pass `true` only when the user asks about finished/delivered shipments; omit to get both.
- **`demurrage_risk` / `detention_risk`**: both `from` and `to` are required (not open-ended like the ETA/ETD ranges). For a relative request like "in the next 5 days," compute the concrete dates yourself before calling — don't pass a relative string.
- **`flags`**: multiple flags are AND'd together (must match all), not OR'd. `HAS_OPEN_TASK`, `HAS_ACTIVE_EXCEPTION`, `HAS_CUSTOMS_HOLD`.
- **`modes` / `statuses`**: these ARE OR'd — matches any of the provided values.
- Pagination: pass the previous response's `end_cursor` as `after` to get the next page; stop when `has_next_page` is false.

## Presenting results

Surface what the user actually asked about first (e.g. lead with demurrage last-free-day if that's why they asked), not the full raw record. Flag open exceptions and customs holds proactively — those are usually the reason someone is checking on a shipment.
