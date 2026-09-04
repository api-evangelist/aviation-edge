---
name: aviation-edge-reference-lookup
description: Resolve airports, airlines, aircraft, aircraft types, cities, countries, routes, taxes and timezones from the Aviation Edge static databases, and search or geolocate them.
api: Aviation Edge Real-Time Reference API
generated: '2026-09-04'
method: generated
source: openapi/aviation-edge-reference-api-openapi.yml, data-model/aviation-edge-data-model.yml
operations:
  - getAirports
  - getAirlines
  - getAircraft
  - getAircraftTypes
  - getCities
  - getCountries
  - getTaxes
  - getTimezones
  - getRoutes
  - autocomplete
  - getNearby
---

# Look up aviation reference data

These eleven operations are the join table for everything else in the API. The dynamic
endpoints return bare IATA and ICAO codes; these turn them into names, coordinates and
countries.

## Steps

1. **From a name or fragment, start with `autocomplete`** — `/autocomplete?key=…&city=ams`
   returns matching cities, airports, rail and bus stations with `code`, `name`, `cityCode`,
   `countryCode`, `lat`, `lng`, `timezone` and `type`.
2. **From a coordinate, use `getNearby`** — `/nearby?key=…&lat=…&lng=…&distance=…`. It returns
   the same location shape plus `distance` from the point you gave, in metres.
3. **From a code, hit the matching database:**
   - `getAirports` `/airportDatabase?codeIataAirport=AAH` — coordinates, timezone, GMT offset, country, city
   - `getAirlines` `/airlineDatabase?codeIataAirline=AA` — callsign, hub, fleet size, fleet age, status
   - `getAircraft` `/airplaneDatabase?numberRegistration=HB-JVE` — one airframe, by registration or `hexIcaoAirplane`
   - `getAircraftTypes` `/planeTypeDatabase?codeIataAircraft=100` — type names
   - `getCities` `/cityDatabase?codeIataCity=AAA`, `getCountries` `/countryDatabase?codeIso2Country=AD`
   - `getTaxes` `/taxDatabase?codeIataTax=AC` — IATA aviation tax codes
   - `getTimezones` `/timezoneDatabase` — airport timezones
4. **For "which airlines fly this route", use `getRoutes`** — filter by `departureIata`,
   `airlineIata` or `flightNumber`. This is static schedule data, not live availability.
5. **Join client-side.** There are no links or expandable references in the responses. Match
   `codeIataCity`, `codeIso2Country`, `codeIataAirline` and `numberRegistration` yourself —
   `data-model/aviation-edge-data-model.yml` lists every join in the graph.

## Rules

- **Omitting every filter returns the entire database.** That is a supported call and costs one
  quota unit, but the airports, airplanes and cities databases are very large; the provider's
  own console warns it can crash a browser. Cache a full pull rather than repeating it.
- The surrogate ids (`airportId`, `airlineId`, `cityId`, `taxId`, `planeTypeId`) are display
  values only — no endpoint accepts them as a filter.
- Coverage is not total: small, military and some private airports and heliports may be
  missing, and the provider asks to be told about gaps rather than claiming completeness.
- This API sells no ticketing, pricing or seat-availability data. If a request needs booking
  information, say so and stop.
