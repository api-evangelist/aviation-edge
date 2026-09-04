---
name: aviation-edge-notam-monitoring
description: Retrieve current and historical NOTAMs for an airport or a Flight Information Region from Aviation Edge.
api: Aviation Edge NOTAMs API
generated: '2026-09-04'
method: generated
source: openapi/aviation-edge-notams-api-openapi.yml, https://github.com/AviationEdgeAPI/Notam-API
operations:
  - getNotams
---

# Retrieve NOTAMs

`GET /notams?key=…` returns notices affecting an airport or an airspace region.

## Steps

1. **Pick exactly one selector.** One of `iata` (airport IATA code), `icao` (airport ICAO code)
   or `location` (FIR location code, e.g. `ZNY`) is required. Calling without one returns
   `{"error":"Either \"icao\" or \"iata\" or \"location\" parameter must be provided","success":false}`
   with HTTP 200.
2. **Add a window for history.** `date_from` and `date_to` (YYYY-MM-DD) select historical
   NOTAMs; omit them for what is currently in force.
3. **Read the structured fields for filtering, the text for meaning.** Each record carries
   `location`, `number` (e.g. `a4247/26`), `class`, `startdateutc`, `enddateutc` and
   `condition`. `condition` is the original, unparsed NOTAM message — abbreviations, Q-codes,
   coordinates and all.
4. **Decide currency from the dates.** `startdateutc` and `enddateutc` are UTC; compare against
   now to say whether a NOTAM is active, upcoming or expired. These are the only UTC timestamps
   in the API — everything else is airport-local.

## Rules

- **Do not paraphrase a NOTAM as operational advice.** Summarise it, quote `condition`, and say
  which authority publishes it. Aviation Edge's own README tells users to verify with the
  relevant aviation authority before acting.
- The API does not parse NOTAM text into structured Annex 15 / AIXM fields. If a question needs
  the runway, altitude band or Q-code, that has to come out of `condition` and should be
  presented as an extract, not as a derived fact.
