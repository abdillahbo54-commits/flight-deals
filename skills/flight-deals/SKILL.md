---
name: flight-deals
description: Find the cheapest and best flights in one exchange, with no setup. Use when the user asks for flights, airfare, cheap flights, plane tickets, cheapest dates, a price comparison, a price alert or a full trip cost, in any language (e.g. "vol pas cher", "billet d'avion", "vol Casablanca Paris en décembre", "cheap flights London to Dubai"). Several flight comparators in one skill. Queries every available flight connector (lastminute.com, eDreams, Expedia…) and, with a browser, Skyscanner and Google Flights, then shows the same flights side by side with each source's total price. Compact top-5 table, never invents a price, never books without explicit approval.
---

# Flight Deals — find the best flights (v1.9)

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
| **Travellers and ages** | |
| "2 adultes", "2 adults", "à deux", "en couple" | 2 adults |
| "avec 2 enfants de 5 et 9 ans", "2 kids (5, 9)" | + 2 children, ages 5 and 9 |
| "avec un bébé de 10 mois", "infant" | + 1 infant (under 2, on lap unless "own seat") |
| "un ado de 14 ans", "teen 15" | 12–17: usually priced as adult; pass the age if the tool accepts it |
| "senior", "65 ans" | adult (pass the age if the tool has a senior type) |
| "en famille" | unclear count: search 2 adults, say so, ask the count after the results |
| **Trip type** | |
| "aller simple", "one way", "sans retour" | one way |
| "aller-retour", "return" | round trip (default) |
| "retour depuis Marrakech", "open jaw" | open-jaw (return from another city) |
| **Stops** | |
| "direct", "sans escale", "non-stop" | non-stop only |
| "1 escale max", "max one stop" | at most 1 stop |
| "escale OK", "peu importe" | any number of stops (default) |
| "escale de moins de 3 h", "short layover" | max layover duration |
| "pas d'escale de nuit", "no overnight layover" | exclude overnight layovers |
| "pas de changement d'aéroport" | exclude airport changes |
| **Stopover** (a planned stay of 1+ days in a city on the way) | |
| "stopover 2 jours à Istanbul", "arrêt à Lisbonne" | search as multi-city: origin → stopover city, then → destination (+ return); show the total |
| "pas de stopover" | no planned stay (default); long layovers are still flagged |
| **Other** | |
| "du 18 au 28 décembre", "18/12 → 28/12" | exact dates, no ±3 days |
| "business", "premium" | cabin |
| "avec bagage", "23 kg", "valise en soute" | checked bag needed: add bag cost to the total |


Prices are always the **total for all passengers**: say "total for 2
adults" in the header line.

Apply the filters to the results, not by guessing: if a tool has a filter
parameter (stops, cabin, passenger ages), use it; otherwise filter the rows it
returned and say how many were removed. If a filter leaves no result, say so
and show the closest options (e.g. "no direct flight; best with 1 stop").

Stopover: some airlines run stopover programmes (a free or cheap stay in their
hub). Mention it only as something to check on the airline's site; never
claim a programme or price you did not see in a tool.

After the results, ask **at most one** clarifying question, the one most
likely to change the ranking (usually: "Exact dates, or a checked bag?").

If origin or destination is missing, ask for it in one short question; this
is the only question allowed before searching.

## 2. Search

Check silently which tools exist, then query in parallel (same turn):

1. **Every connected flight tool** (names like `search_flights`,
   `flight_lookup`, `booking_options`). One query per source; use city codes.
   **Ask each tool for 5 results** (`max_results=5`, `limit=5`…): some tools
   require showing every row they return, so asking for 5 keeps the table
   compact without breaking their rules. More rows only on request.
2. **Browser sources** — only if a browser tool (Claude in Chrome or
   equivalent) is connected:
   - **Skyscanner** (fast: one URL, no form). Procedure:
     `references/sources.md` § Skyscanner.
   - **Google Flights** (form only; a `?q=` URL opens the home page without
     searching). Procedure: `references/sources.md` § Google Flights.

   **Always check both, without being asked**, on every search, whenever a
   browser is connected: Skyscanner first (one URL, fast), then Google
   Flights. Run them right after launching the connector queries. The
   comparison across sources is part of the answer, not an option.
   Skip a browser source only if it is blocked (captcha, consent wall,
   error) or no browser exists, and say which.

   **Which dates to check in the browser** (flexible dates): the dates the
   user asked for **and** the most interesting date combination the
   connectors found (lowest total, or the only direct option), if
   different. At most these 2 date combinations per browser source, to keep
   the search fast. State which dates each source covered.

   Whenever a browser source is not checked, **say why** in the sources line,
   with one of:
   - "Google Flights: not checked (no browser connected)";
   - "Google Flights: not checked (captcha / consent / error: …)".
   The same wording applies to Skyscanner.
3. **Web search** only for route facts (airlines, baggage rules, season).
   Never take prices from snippets.

### Robustness

- Each source gets **one retry** on error (same query). If it fails again,
  write one line with the **tool's own error text**, quoted, and continue:
  "Expedia: not checked (error `LOCATION_REQUIRED`)". Do not guess a cause
  (country block, outage…) the error does not state. If the error says what
  is missing (e.g. the user's location), mention it once so the user can
  provide it; never invent the value.
- **Region-locked tools** (e.g. `GEOFENCING_ERROR`, "not available for your
  location"): do not retry. Say it is unavailable **from the user's region,
  not a temporary outage**, and continue with the other sources.
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
   **If a tool forbids rankings or opinions** (lastminute.com does), do not
   write "cheapest / best" for its rows. Give **facts** instead: which rows
   are direct, total durations, long or overnight layovers, odd arrival
   airports (e.g. XNB = a bus terminal, not an airport), bags unknown.
   This applies **everywhere in the answer**, including the opening summary:
   write "only direct round trip found, 1849,86 €", never "the lowest total
   I found" about that tool's offers. Comparing the same flight across
   sources (section 4, comparison table) is a fact and stays allowed.
3. Apply `references/regional-tips.md` (general rules + the route's region):
   peaks, baggage, alternate airports, hub connections.
4. **Match the same flight across sources**: same airline (or operating
   carrier), same departure time, same stops. These rows feed the
   comparison table (section 4).

## 4. Present

**Compact first**: **at most 5 rows per table** and at most 3 tables (main
results, flexible dates, source comparison). For flexible dates, show the 5
date combinations with the lowest total; the rest only on request ("full
list"). If a tool requires showing every row it returns, ask it for fewer
rows (one call per date combination with 1 result each, then keep 5).
Then one line
per pick (cheapest / fastest / best balance, with the reason) or the factual
notes when a tool forbids picks, the search time, and "fares move quickly".

**Answer layout, in this exact order** (nothing after step 7):
1. one header line (route, dates, travellers, what prices include);
2. the tables (max 5 rows each, max 3 tables);
3. the factual notes ("À savoir"), the sources line and the tracking-link
   notice;
4. the tool's booking call-to-action, if it requires one;
5. the tool's mandatory offer, if any (lastminute.com: "Would you like me to
   re-sort by 'recommended' (best overall value)?", in the user's language);
6. the **settings line**, so a beginner knows what can be changed (format
   below);
7. **one** clarifying question, last: a single question about a single
   thing, the one most likely to change the real price (usually the checked
   bag, or exact dates if the user gave only a month). Never two questions
   joined by "and" / "et".

**No summary or repeated conclusion** anywhere after the tables: say each
fact once.

**Several dates from one tool** (flexible dates, calendar confirmation):
keep the tool's own table and headers. Put a short date line above each small
table ("**Sam. 19 déc.**"), at most 3 dates; never write the date into the
tool's cells (e.g. not "Transavia (19 déc.)") and keep the tool's row
numbers. Other dates go only in the comparison table.

Settings line format, in the user's language:

> Change: travellers and ages (e.g. "2 adults + 1 child of 6") · one way /
> round trip · stops (direct / 1 max / no overnight) · stopover (e.g.
> "2 days in Istanbul") · exact dates · checked bag · full list

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

### Source comparison (always, when 2+ sources answered)

After the main table, add a short **comparison table** for the flights found
on at least two sources (up to 5 rows), with the **total for all
travellers** from each source:

(Example with real totals from a CMN → DXB test, 6 Nov 2026.)

| Flight | lastminute.com | Skyscanner | Google Flights |
|---|---|---|---|
| Qatar Airways 13:00 · 1 stop | 1082,40 € | 976 € | 1 066 € |
| Royal Air Maroc 01:30 · direct | 1202,21 € | 1 085 € | 1 222 € |

- Copy each price exactly as that source shows it; "—" when not found.
- Facts only: no "best deal" wording. One line under it may state the
  lowest total per flight ("Qatar 13:00: lowest on Skyscanner, 976 €").
- Browser prices: report the **total for all travellers** (Skyscanner shows
  per-person and total; Google Flights shows the total for the selected
  passengers: check the page wording). Never multiply a per-person price
  yourself if the site gives a total.
- Give each browser source's results page URL as its link. If the browser
  tool hides or redacts the URL (e.g. Google Flights `tfs=REDACTED`), do not
  rebuild it: leave the results tab open and say "results page left open in
  your browser" instead of a link.
- **Ignore sponsored rows** ("Sponsorisé", "Sponsored", "Publicité par…",
  "Ad"): they often show a different price for the same flight. Use only the
  organic results; if you mention an ad, label it as an ad.

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
