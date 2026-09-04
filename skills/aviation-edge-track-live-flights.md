---
name: aviation-edge-track-live-flights
description: Track live flights worldwide with the Aviation Edge Flights Tracker API — by flight number, airline, route, or a radius around a coordinate.
api: Aviation Edge Real-Time API
generated: '2026-09-04'
method: generated
source: openapi/aviation-edge-real-time-api-openapi.yml, conventions/aviation-edge-conventions.yml, errors/aviation-edge-problem-types.yml
operations:
  - getRealTimeFlights
  - getAirlines
  - getAirports
---

# Track live flights

Base URL `https://aviation-edge.com/v2/public`. Every call is a GET and carries your
subscription key as `key`. There is no free or test key.

## Steps

1. **Resolve the codes first.** Aviation Edge filters on IATA codes, not names. If you were
   given an airline or airport name, resolve it with `getAirlines`
   (`/airlineDatabase?key=…&codeIataAirline=…`) or `getAirports`
   (`/airportDatabase?key=…&codeIataAirport=…`), or use `autocomplete` for a partial string.
2. **Call `getRealTimeFlights`** — `GET /flights?key=…` with exactly the filter that matches
   the question:
   - one flight: `flightIata=W8519`
   - one airline's fleet in the air: `airlineIata=W8`
   - departures from an airport: `depIata=MAD`; arrivals: `arrIata=GIG`
   - a geographic box: `lat=51.5074&lng=0.1278&distance=100`
   - always add `limit` (max 30000) when you do not filter, or you will pull every live
     flight on earth into the context.
3. **Read the response.** Each element carries `aircraft`, `airline`, `departure`, `arrival`,
   `flight`, `geography` (altitude, direction, latitude, longitude), `speed`, `status` and
   `system.updated` (a Unix timestamp of the last position fix).
4. **Poll, do not subscribe.** There are no webhooks and no streaming surface; the provider's
   FAQ says to compare successive calls yourself. Positions refresh roughly every 5 minutes,
   so polling faster than that only burns quota.

## Rules

- **A 200 is not a success.** Check `success` in the body before parsing. `{"message":"Missing
  API Key","success":false}` and `{"error":"Invalid API Key","success":false}` both arrive
  with HTTP 200. See `errors/aviation-edge-problem-types.yml`.
- **A flight that has not taken off is not here.** The tracker only returns airborne flights,
  and there is a few-minute lag after wheels-up. For scheduled or landed flights use
  `getTimetable` instead (see the airport-schedules skill).
- **Quota is monthly, not per second**, and no header reports what is left; one call returning
  30,000 records costs the same one call. Errors do not consume quota.
- **Never use this data for navigation or traffic advisories.** The provider's terms and SLA
  exclude operational use, and only static data is covered by the SLA at all.
