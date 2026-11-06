# Changelog

## 1.1.0 — 2026-11-06

### Added
- Travellers, one way, exact dates, bag, direct-only and cabin are read from
  the user's sentence (any language, any order); table in `SKILL.md`.
- A short settings line under the results so beginners see what they can
  change ("2 adults + 1 child · one way · exact dates · bag…"); a plain-words
  reply re-runs the search.
- README section "Travellers, one way, dates…" (EN and FR).

## 1.0.0 — 2026-11-06

First public release (renamed from the internal draft "searchfly").

### Added
- Quick mode: origin + destination + period starts the search immediately
  (1 adult, economy, round trip, ±3 days); at most one question afterwards.
- Grouping of same-price offers that differ only by departure time.
- Compact top-5 table first, full detail on request.
- Tool display rules kept in the condensed table (exact price,
  `table_headers`, unmodified deeplink, neutral tone, `ranking_best` re-sort).
- One retry per source, then a one-line failure notice.
- Notice that partner booking links may contain tracking parameters.
- `references/iata-codes.md`: city and airport codes, "prefer the city code".
- `references/regional-tips.md`: worldwide tips by region (peaks, baggage,
  alternate airports, hub connections, transit visas).
- French README.

### Changed
- Google Flights: `?q=` URLs removed (they open the home page without
  searching). Replaced by a tested form-filling procedure; Google Flights is
  now optional.
