# Changelog

## 1.8.0 — 2026-11-06

Fixes from a round-trip, flexible-dates test (CMN → DXB, ±1 day).

### Fixed
- No ranking words about a tool's offers anywhere in the answer when that
  tool forbids it (the summary said "the lowest total I found" about a
  lastminute.com offer). Facts only; cross-source comparison stays.
- Browser sources now check the requested dates and the most interesting
  date combination found by the connectors (it was skipped in the test).
- Shorter answers: at most 5 rows per table, at most 3 tables; flexible
  dates show the 5 lowest combinations, full list on request.

## 1.7.0 — 2026-11-06

Fixes from a 3-source test (lastminute.com, Skyscanner, Google Flights).

### Fixed
- Sponsored rows (Skyscanner "Sponsorisé / Publicité") are ignored: they
  showed another price for the same flight.
- When the browser tool redacts the Google Flights results URL, the tab is
  left open and the answer says so; the URL is never rebuilt.
- The end-of-answer order is now unambiguous: booking call-to-action,
  re-sort offer, settings line, then one question.

### Added
- Google Flights procedure: passengers dialog and one-way selection (tested).

## 1.6.1 — 2026-11-06

### Changed
- Positioning: "several flight comparators in one skill" (skill
  description, READMEs, banner).
- New README image: the same 5 flights compared on lastminute.com,
  Skyscanner and Google Flights (real totals, CMN → DXB, 3 travellers).

## 1.6.0 — 2026-11-06

### Changed
- Skyscanner and Google Flights are checked on **every** search when a
  browser is connected, without being asked.
- New source-comparison table: the same flights side by side with each
  source's total price (copied exactly, "—" when not found).

## 1.5.0 — 2026-11-06

### Added
- Skyscanner as a browser source, with a tested direct-URL procedure
  (route, date, travellers and ages, cabin, stop filter). Reports the total
  price for all travellers.
- Automatic browser check (Skyscanner, then Google Flights) when fewer than
  2 connectors return flights, e.g. when Expedia is region-locked.
- Trigger phrases: "vérifie aussi Google Flights / Skyscanner".

## 1.4.0 — 2026-11-06

### Changed
- When Google Flights is not checked, the answer says why (no browser,
  skipped, captcha / error) and offers to check it when a browser exists.
- Region-locked tools (`GEOFENCING_ERROR`) are reported as unavailable from
  the user's region, not as an outage, and are not retried.
- `sources.md`: Expedia can be region-locked (seen from Morocco).

## 1.3.0 — 2026-11-06

Fixes from a real test (CMN → DXB, one way, 2 adults + 1 child, max 1 stop).

### Fixed
- Tables stay compact: each tool is asked for 5 results, because some tools
  require every returned row to be shown (the test showed 15 rows).
- The tool's mandatory "re-sort by recommended" offer is always included.
- Failed sources are reported with the tool's own error text; no guessed
  cause (the test said "blocked from Morocco" for a `LOCATION_REQUIRED` error).
- Every answer ends with one clarifying question (usually the checked bag).

### Changed
- When a tool forbids rankings, the skill gives factual notes (direct
  flights, durations, overnight layovers, odd arrival airports) instead of
  "cheapest / best" picks.

## 1.2.0 — 2026-11-06

### Added
- Ages of travellers (children, infants, teens, seniors), passed to tools
  that accept them.
- Stop filters: direct only, max 1 stop, max layover time, no overnight
  layover, no airport change. Empty results explained, closest options shown.
- Stopover: a planned stay in a city on the way, searched as multi-city.
- Settings line and READMEs list these options.

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
