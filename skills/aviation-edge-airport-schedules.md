---
name: aviation-edge-airport-schedules
description: Pull an airport's departure or arrival timetable from Aviation Edge for now, for a past date range, or for a future date — including delays, gates and cancellations.
api: Aviation Edge Real-Time Schedules API
generated: '2026-09-04'
method: generated
source: openapi/aviation-edge-schedules-api-openapi.yml, openapi/aviation-edge-real-time-api-openapi.yml, conventions/aviation-edge-conventions.yml
operations:
  - getTimetable
  - getHistoricalFlights
  - getFutureFlights
---

# Read an airport timetable

Three operations answer the same question for three time windows. Pick by the date asked for.

| When | Operation | Endpoint | Required |
|---|---|---|---|
| Now (about -6h to +6h) | `getTimetable` | `/timetable` | `iataCode`, `type` |
| A past date or range | `getHistoricalFlights` | `/flightsHistory` | `code`, `type` |
| A future date | `getFutureFlights` | `/flightsFuture` | `type`, `iataCode`, `date` |

## Steps

1. **Choose departures or arrivals.** `type=departure` or `type=arrival` is mandatory on all
   three; there is no combined response, so ask for both separately if you need both.
2. **Mind the parameter names — they differ between the three.** The real-time and future
   endpoints take `iataCode`; the historical endpoint takes `code`. The historical endpoint
   uses `date_from` / `date_to`, the future endpoint uses a single `date`.
3. **Narrow before you pull.** `airline_iata` (real-time and historical), `flight_number` and
   `status` reduce a whole-airport timetable to the flights you care about.
4. **Respect the windows.** Historical ranges are capped at 30 days per call and one year of
   history by default; future schedules run up to a year ahead but not within the next 7 days.
   For a longer historical span, make several calls.
5. **Read delays as minutes.** `departure.delay` / `arrival.delay` are minutes; the schedule
   objects also carry `scheduledTime`, `estimatedTime`, `actualTime`, `estimatedRunway`,
   `actualRunway`, `gate`, `terminal` and `baggage`. All times are the airport's local time.

## Rules

- `getHistoricalFlights` and `getFutureFlights` are the only operations that return **HTTP 400**;
  they answer `{"message": {"type": "Missing param: type (str)"}}` when a required parameter is
  missing. Every other failure comes back as HTTP 200 with `success: false`.
- Gate and baggage data depend on the airport and are frequently null. Do not present a null
  gate as "no gate assigned".
- Future schedules are generated from historical data by an algorithm; treat them as a
  projection, not as a published timetable.
