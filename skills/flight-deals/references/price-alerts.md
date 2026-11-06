# Price tracking

Use when the user says "alert me", "tell me if it drops", "watch this route",
or when waiting is a real option (departure is months away).

## Setup (confirm in one message)

Collect: route, dates or window, passengers, cabin, bag needs, **target price**
(or "any drop of X %"), how often to check, how long to track, and where to be
notified.

If no target price is given, propose one: about 10 % under the best price found
today, and a weekly check.

## How

1. Run a baseline search now and record: date, best price, source, itinerary.
2. Create a scheduled task (the session's scheduling tool) whose prompt is
   fully self-contained: route, dates, passengers, filters, baseline price,
   target, the sources to check, and the reporting rule below. Each run starts
   with no memory of this conversation.
3. Tell the user the schedule, the target and how to stop it.

If no scheduling tool is available, say so and offer a manual re-check command
the user can ask for ("re-check my Paris → Bangkok watch").

## Reporting rule for each run

- Report **only** if the best price is below the target, or at least 5 %
  lower than the last reported price, or the route sells out / jumps sharply.
- Otherwise stay silent or send a one-line "no change".
- Always include: new price, previous price, source, link, search timestamp.
- Never book automatically.

## Booking-timing advice (give as guidance, not certainty)

- Short-haul: prices often rise in the last 2–3 weeks.
- Long-haul and peak season: book earlier; waiting is riskier.
- Prices can rise after a drop: if the price is under the user's target, say so
  plainly rather than urging them to wait.
