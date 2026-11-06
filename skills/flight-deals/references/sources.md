# Sources registry

Flight Deals uses whatever is available. Order of preference: connected tools
(structured, reliable) → browser on public search pages → web search for facts.

## A. Named services (built in)

| Service | Access | Flights | Hotels | Notes |
|---|---|---|---|---|
| lastminute.com | Connector (`search_flights`, hotels, packages) | ✅ | ✅ | Display rules: exact price, `table_headers`, unmodified deeplink, neutral table, offer `ranking_best=True` re-sort |
| eDreams | Connector (`search_flights`, `search_hotels`) | ✅ | ✅ | Strong on European routes; check final price with fees |
| SlickTrip | Connector (flight lookup, booking options, price-drop alerts) | ✅ | ✅ | Account/login required; best for alerts and seat info |
| Skyscanner | Browser (direct URL, tested Nov 2026) | ✅ | ✅ | Fastest browser source: one URL with route, date, travellers, stops (see section H). Shows per-person and total prices; may show a captcha |
| Trip.com | Browser (public search pages) | ✅ | ✅ | Good on Asia routes and long-haul combos; check bag rules |
| Agoda | Browser (public search pages) | ➖ | ✅ | Hotel focus (strong in Asia); use for hotel/package price checks |
| Expedia | Connector (`search_flights`, `search_hotels`) | ✅ | ✅ | Region-locked in some countries (seen from Morocco: `GEOFENCING_ERROR`, also `LOCATION_REQUIRED`): report as unavailable from the user's region, no retry. "Unknown error" = connection expired: retry once, then report |
| Google Flights | Browser (optional, form only) | ✅ | ➖ | Reference price, price calendar; fill the form, `?q=` URLs do not search (see section G) |
| Kayak / Momondo | Browser | ✅ | ✅ | Cross-check, flexible dates |
| Booking.com, Trivago | Connector (hotels) | ➖ | ✅ | Same-dates hotel comparison |
| Airline sites | Browser | ✅ | ➖ | Final price, fare families, bag rules |

Access types:
- **Connector**: call the tool directly. If it is not installed in the session,
  say which connector would help and continue with the others.
- **Browser**: open the public search page with the trip parameters, read the
  results, and report them. No login bypass, no captcha solving, no scraping at
  scale. If blocked, say so and move on.

## B. Generic detection

Any tool whose name looks like `search_flights`, `flight_lookup`,
`booking_options`, `search_hotels`, `accommodations_search` or
`search_flight_and_hotel_package` is used automatically, even if the service is
not in the table above.

## C. Query plan (to limit time and cost)

1. All connectors in parallel, with **city codes** (PAR, LON, NYC…).
2. Google Flights in the browser if one is connected (optional).
3. For the **top 3 candidates only**, cross-check in the browser on
   Skyscanner, Trip.com and the airline site.
4. For hotels: connectors first, then Agoda / Booking.com for the same dates.
5. Label every price with its source and the search time.
6. Errors: one retry per source, then report in one line and continue.

## D. Fact sources (not prices)

- Airline route maps and baggage policy pages
- Official government pages for visa / transit rules
- Weather, season and events at the destination (price drivers)

## E. Output / storage tools (optional)

| Tool type | Use |
|---|---|
| Google Drive / Sheets, Notion, Docs | Save the shortlist and comparison |
| Calendar | Tentative hold on chosen dates |
| Scheduled tasks | Recurring price check (see price-alerts.md) |
| Email draft | Send the shortlist to a travel companion |

## F. Adding a new source

1. Add a row in the table in section A (service, access type, flights/hotels, notes).
2. Connector: nothing else to do, it is detected automatically (section B).
3. Website: add the URL pattern and what to read on the results page.

## G. Google Flights — browser procedure (tested Nov 2026)

`google.com/travel/flights?q=Flights from X to Y on DATE…` and
`/travel/flights/search?q=…` both open the **home page without searching**.
Do not use them. Fill the form instead:

1. Open `https://www.google.com/travel/flights?hl=<lang>&curr=<CUR>`.
2. Trip type, passengers, cabin: change the three dropdowns above the form
   only if they differ from the request (default: round trip, 1, economy).
3. **Origin** ("Where from?" / "De"): click, select all (Ctrl/Cmd+A), type
   the IATA code, wait ~1 s, press Enter to take the first suggestion.
4. **Destination** ("Where to?" / "À"): type the city code (e.g. `PAR`),
   wait, Enter. The first suggestion is the city with all its airports.
5. **Dates**: click the departure field; a calendar dialog opens showing a
   **price per day** (useful for flexible dates: read it before closing).
   Inside the dialog, click its departure textbox, type the date in the
   page's locale format (FR: `18/12/2026`, EN-US: `12/18/2026`), Tab; same
   for the return textbox; click **Done / OK**.
6. Check the form shows the right cities and dates, then click
   **Search / Rechercher**.
7. Wait 5–7 s, then read the page text. If it shows only "Search results"
   with no flights, wait a few seconds and read again.
8. Read each row: times, airline, duration, airports, stops, price (round
   trip, all passengers, taxes included, **bags not included**).
9. The results URL (`/travel/flights/search?tfs=…`) is shareable: give it as
   the Google Flights link. Some browser tools redact it (`tfs=REDACTED`):
   then leave the tab open and say so; never rebuild the URL.
10. Passengers: open the passengers button, use "+" for adults and for
    children 2–11 / infants, then the dialog's OK button (tested). Trip
    type: the "Aller-retour / Round trip" dropdown → "Aller simple / One way".

Ignore the user's "Tracked prices" section and any personal history on the
page. If a consent screen appears, decline non-essential cookies; if a
captcha or login wall appears, stop and report.

## H. Skyscanner — browser procedure (tested Nov 2026)

Unlike Google Flights, Skyscanner accepts the search directly in the URL:

```
https://www.skyscanner.fr/transport/vols/<from>/<to>/<YYMMDD>/[<YYMMDD return>/]?adultsv2=<n>&childrenv2=<age>|<age>&cabinclass=economy&rtn=<0|1>[&stops=…]
```

- `<from>` / `<to>`: lowercase IATA codes. Skyscanner's own city codes
  cover all airports (e.g. `dxba` = Dubai all airports, `pari` = Paris all,
  `lond` = London all). If unsure, use the main airport code (`cdg`, `dxb`).
- Dates: `YYMMDD` (18 Dec 2026 = `261218`). Return date as a second path
  segment, with `rtn=1`. One way: one date, `rtn=0`.
- `adultsv2=2`; `childrenv2=9` or `childrenv2=5|9` for several children
  (ages, pipe-separated; infants are age 0 or 1).
- `cabinclass=economy|premiumeconomy|business|first`.
- Stops: `stops=!oneStop,!twoPlusStops` = direct only;
  `stops=!twoPlusStops` = max 1 stop (tested). Omit for any.
- Use the site for the user's language (`skyscanner.fr`, `.net`, `.es`…).

Then:
1. Open the URL, wait 6–8 s, read the page text.
2. If it still says "Loading results… (n of m providers)", wait a few more
   seconds and read again, so the cheapest offers are in.
3. Each result shows a **per-passenger price** and a **"Total" / "Montant
   total"**: report the **total**, and say it is the total for all travellers.
4. The results page URL is the link to give the user.
5. **Skip sponsored rows** ("Sponsorisé", "Publicité par …", "Réservez avec
   <airline> à partir de"): same flight, different price. Use organic rows.
6. "Combinaisons de compagnies" / "airline combinations" can mean separate
   tickets: flag it as a self-transfer risk to check.

If a captcha ("press and hold", "are you a robot") appears, stop: never solve
it. Report "Skyscanner: not checked (captcha)" and continue.
