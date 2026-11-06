---
name: flight-deals
description: Find the cheapest and best flights in one exchange, with no setup. Use when the user asks for flights, airfare, cheap flights, plane tickets, cheapest dates, a price comparison, a price alert or a full trip cost, in any language (e.g. "vol pas cher", "billet d'avion", "vol Casablanca Paris en décembre", "cheap flights London to Dubai"). Searches every available flight connector, optionally Google Flights in the browser, groups duplicate fares, shows a compact top-5 table, never invents a price and never books without explicit approval.
---

# Flight Deals — find the best flights (v1.1)

Goal: a beginner types "vol Casablanca Paris en décembre" and gets a clear
table **in one exchange**. Reliability first: every price comes from a tool,
nothing is booked without a clear yes.

References (read only when needed):
- `references/iata-codes.md` — city and airport codes
- `references/sources.md` — sources, Google Flights browser procedure
- `references/comparison-checklist.md` — real-total-price rules
- `references/price-alerts.md` — price tracking
- `references/regional-tips.md` — tips by region (peaks, bags, alternate airports, hubs)

Reply in the user's language.

## 1. Quick mode (default)

If the message gives **origin + destination + a period**, search immediately.
Do not ask anything before the first results.

Defaults (state them in one line above the table):

| Field | Default |
|---|---|
| Passengers | 1 adult |
| Cabin | economy |
| Trip | round trip |
| Dates | flexible ±3 days around the period; for a month only ("en décembre"), pick a representative 7–10 night window in that month and say so |
| Currency | the user's currency if known, else the tool's default |
| Codes | **city code** when the city has several airports (PAR, LON, NYC…), see `references/iata-codes.md` |

### Read the trip details from the message

Anything the user writes overrides a default. Read it in any language and
any order, for example:

| The user writes | Meaning |
|---|---|
| "2 adultes", "2 adults", "à deux", "en couple" | 2 adults |
| "avec 2 enfants (5 et 9 ans)", "2 kids" | + 2 children (ages matter: ask only if a tool requires them) |
| "avec un bébé", "infant" | + 1 infant on lap |
| "en famille" | unclear count: search 2 adults, say so, and ask the count after the results |
| "aller simple", "one way", "sans retour" | one way |
| "aller-retour", "return" | round trip |
| "du 18 au 28 décembre", "18/12 → 28/12" | exact dates, no ±3 days |
| "retour depuis Marrakech", "open jaw" | open-jaw (return from another city) |
| "business", "premium" | cabin |
| "avec bagage", "23 kg", "valise en soute" | checked bag needed: add bag cost to the total |
| "direct", "sans escale" | non-stop only |

Prices are always the **total for all passengers**: say "total for 2
adults" in the header line.

After the results, ask **at most one** clarifying question, the one most
likely to change the ranking (usually: "Exact dates, or a checked bag?").

If origin or destination is missing, ask for it in one short question; this
is the only question allowed before searching.

## 2. Search

Check silently which tools exist, then query in parallel (same turn):

1. **Every connected flight tool** (names like `search_flights`,
   `flight_lookup`, `booking_options`). One query per source; use city codes.
2. **Google Flights in the browser — optional.** Only if a browser tool
   (Claude in Chrome or equivalent) is connected. Fill the form; never rely
   on a `?q=` URL (it opens the home page without searching). Procedure:
   `references/sources.md` § Google Flights. If no browser is available,
   skip it and say "Google Flights not checked (no browser)".
3. **Web search** only for route facts (airlines, baggage rules, season).
   Never take prices from snippets.

### Robustness

- Each source gets **one retry** on error (same query). If it fails again,
  write one line ("Expedia: unavailable — Unknown error, connector may need
  reconnecting") and continue with the others.
- Login wall, captcha, bot check or consent wall you cannot decline: stop on
  that source, say so, move on. **Never bypass** connections, captchas or
  site protections, and never ask the user for passwords.
- If every source fails, say so plainly and give the user the Google Flights
  home page to search themselves. Never fill the gap with estimated prices.

## 3. Group and rank

1. **Group duplicates**: offers with the same airline(s), same route/airports,
   same stops and the **same price**, differing only by departure times,
   become one row. Show the times as a list ("06:10 · 11:40 · 17:25 ·
   21:05") and keep the link of one of them (say which time it books).
2. Rank by total price for all passengers, then flag cheapest, fastest,
   best balance (see checklist for bags, layovers, risk).
3. Apply `references/regional-tips.md` (general rules + the route's region):
   peaks, baggage, alternate airports, hub connections.
4. If the same flight appears on several sources at different prices, keep
   the cheapest row and note the gap ("also 214 € on Google Flights").

## 4. Present

**Compact first**: one table, **top 5 rows** after grouping, then one line
per pick (cheapest / fastest / best balance, with the reason), the search
time, and "fares move quickly". Then add one short **settings line** so a beginner knows what can
be changed, in the user's language, e.g.:

> Change: travellers (e.g. "2 adults + 1 child") · one way · exact dates ·
> checked bag · direct only · full list

The user can answer in plain words ("à deux, aller simple"): re-run the
search with the new settings, no further questions.

Columns: airline · route (airports) · times · duration · stops · price ·
source · link.

### Tool display rules (always win over compactness)

When a connector supplies display instructions, follow them even in the
condensed table. In particular for lastminute.com `search_flights`:
- copy the **price exactly** as returned (amount, currency, format);
- use the tool's **`table_headers`** for column and link labels;
- use the **deeplink URL unmodified** (no trimming, no re-encoding);
- keep a **neutral tone inside the table** (no "great deal!", no emoji);
- after the table, offer to **re-sort by "recommended"** (re-run with
  `ranking_best=True`).

If two tools impose different headers, use one table per source rather than
breaking either tool's rules.

### Transparency on links

Booking links from connectors may contain **tracking or affiliate
parameters**. Say so once, in one line under the table ("Booking links come
from the partner sites and may include tracking parameters"). Never remove
or alter them (the tools require unmodified URLs); if the user prefers a
link without tracking, point them to the airline's own site or the Google
Flights results page instead.

## 5. Optional extensions (offer in one line, do on request)

- **Hotels / packages** for a full trip cost.
- **Price tracking** via a scheduled task (`references/price-alerts.md`).
- **Save the shortlist** to a Sheet / Doc / Notion page / markdown file.
- **Calendar hold** on the chosen dates.

## 6. Rules

- **Never book or pay** without explicit confirmation; never type payment,
  passport or login details.
- **Never invent** a price, schedule or availability. Every price shows its
  source; if a price is "from" or excludes bags, say so.
- Say "cheapest found", never "cheapest anywhere".
- Do not read or reuse the user's personal data shown on a site (tracked
  prices, history, saved trips) unless they ask.
- Facts and trade-offs, no sales pressure.

## Example

User: "vol Casablanca Paris en décembre"

> Searched CMN → PAR (all Paris airports), 18 → 28 Dec 2026, 1 adult,
> economy, round trip, ±3 days. Searched at 14:05.
>
> | Airline | Route | Times (out) | Duration | Stops | Price | Source | Link |
> |---|---|---|---|---|---|---|---|
> | … | CMN–ORY | 05:50 · 18:15 | 3h05 | direct | … | … | … |
>
> Cheapest: … · Fastest: … · Best balance: …
> Expedia: unavailable (Unknown error). Booking links may include tracking
> parameters. Fares move quickly.
>
> Exact dates, or a checked bag needed?
