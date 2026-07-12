# sitrep

**One map, one panel: a live drone-preflight SITREP for wherever the
crosshair points. Keyless, single-file, no backend.**

A self-contained HTML dashboard: a dark map with a centre crosshair and a
docked **SITREP** panel (a bottom sheet on mobile). The panel holds the
**canifly flyability chart** — can a small drone fly *here, now, and for the
next three hours, and how high* — plus a settings box (max wind, range) and a
stack of distance-sorted intelligence cards for everything inside the range
ring: low manned traffic, NWS warnings, FAA restrictions, park no-fly land,
severe-weather outlooks, space weather, and the local forecast.

No backend. No build step. No API keys. No `npm install`. **Open `index.html`
directly in a browser.** Fully responsive — iPhone, iPad, and desktop.

## Run it

```
open index.html      # or just double-click it
```

MapLibre GL loads from CDN; every data feed is fetched client-side from
keyless public APIs.

## The flyability chart (canifly, integrated)

A grid of altitudes (400 ft → surface, 50 ft steps — the FAA Part 107 band)
× four time columns (NOW, +1 h, +2 h, +3 h). Each cell is binary: green =
fly, red = can't. Every gate that can ground or cap a column shows its own
code in the cell where the limit bites:

| Code | Gate | Source |
|---|---|---|
| WIND | wind aloft ≥ your max-wind setting (interpolated 10/80/120/180 m) | Open-Meteo |
| GUST | surface gust ≥ max wind | Open-Meteo |
| VIS | visibility < 3 SM (Part 107) | Open-Meteo |
| SKY | cloud base − 500 ft clearance caps the ceiling | Open-Meteo (pressure-level decks + LCL) |
| KPx | planetary Kp ≥ 5 (GPS/compass risk), forecast bin per hour | NOAA SWPC |
| FAA | LAANC grid ceiling caps the band; a National-Defense TFR in range grounds | FAA ArcGIS (point + range buffer) |
| PROH / NSUF / PARK | Prohibited area, security UAS zone, or NPS land under the crosshair grounds | FAA ArcGIS · NPS |
| PCPN | NEXRAD echo inside the range ring grounds NOW | Iowa State Mesonet mosaic (pixel-sampled) |
| TRFC | manned aircraft in range under 1,000 ft AGL reds its altitude row in NOW | ADS-B |

**Fail-safe posture:** a feed that can't be verified never silently reads
"clear". Unknown inputs fail *open* for the grid values but amber the NOW
header (hover for which inputs are unverified) and the SITREP title. The FAA
airspace query hard-throws on anything that could understate a restriction
(wrong units, truncated results, missing ceilings).

The bold altitude label marks the highest currently flyable row. The
**settings box** below the chart sets **max wind** (±1 mph) and **range**
(±0.5 mi, 0.5–15) — both persisted per device. Range drives the FAA query
radius, the traffic/radar sweep, the SITREP card radius, and the dashed
**range ring** drawn around the crosshair. Zoom is capped so the ring never
exceeds ⅓ of the view.

## The SITREP cards

Everything is measured from the **crosshair** (map centre) and limited to the
range ring. Cards are tiered — red hazards, amber hazards, then routine
traffic — and distance-sorted within each tier (capped at 14, plus pinned and
context cards):

- 🛩️ **LOW AIRCRAFT** (pinned top) — any plane in range under 1,000 ft AGL
  (AGL = barometric altitude − ground elevation at the crosshair). The same
  deduped collector feeds the chart's TRFC cells, so card and chart always
  agree.
- 🚨 emergency squawks (7700/7600/7500) · ⚠️ NWS **warnings** (not watches) ·
  ⛔/⚠️ FAA restrictions (defense, prohibited, security, MOA, stadium…) ·
  🏞️ NPS land · 🗼 controlled airspace (Class B/C/D/E — neutral
  authorization info when you're near it, but when you're **overhead** it the
  card turns red "no-fly" / amber "ceiling N ft" to match the chart's LAANC
  ceiling) · ✈️ / 🪖 civil and military traffic
- Context cards at the bottom: 🌪️/⛈️ SPC Day-1 outlooks, 🧲 geomagnetic
  storms (Kp ≥ 5), ☀️ M/X solar flares, and the 🌡️ local forecast (always
  present).

Tap any card to fly the map there (polygons fly to their nearest edge — the
same edge distance used for sorting and range inclusion). The **SITREP title**
is the one-glance verdict: red when the NOW column is grounded or a red hazard
is in range, amber for a reduced ceiling, an amber hazard, or any unverified
gate input.

## Map layers

| Layer | Source | Refresh |
|---|---|---|
| Aircraft (250 nm around centre, civil) | airplanes.live → adsb.fi fallback | on move · 30 s |
| Military aircraft (worldwide) | ADS-B `/mil` | 30 s |
| Emergency squawks (worldwide) | ADS-B `/squawk` | 30 s |
| NWS warning polygons (US) | api.weather.gov | 30 s |
| NEXRAD reflectivity mosaic | Iowa State Mesonet | per mosaic minute |
| SPC Day-1 outlook (hidden; feeds cards) | NOAA SPC | 15 min |
| FAA airspace + restrictions (in view, ≥z7) | FAA ArcGIS ×5 services | 15 min |
| NPS lands (in view, ≥z7) | NPS ArcGIS | 30 min |

Viewport-scoped layers refetch on every pan/zoom; zoom-gated layers keep
their last data when you zoom out (so a restriction under the crosshair never
vanishes from the chart). Every loader retries with backoff and a failure
never wipes what's on the map.

**Dossier:** right-click (desktop) or long-press (touch) anywhere →
reverse-geocoded place (Nominatim) + country profile (RestCountries) + head of
state (Wikidata) + Wikipedia summary.

## Location & privacy

On load: instant IP fix (ipwho.is), refined by browser GPS if you allow the
prompt (denial is silently tolerated), then one clean camera move framing 3×
the range ring. The ⌖ button recentres on you. Position is resolved
client-side and never leaves the browser except as anonymous lat/lon query
parameters to the public weather APIs.

## Design notes & honest caveats

- **Advisory, not authoritative.** This is a preflight *aid*. LAANC grids,
  TFRs, and NOTAMs change; verify with official sources before flying.
- US-centric hazard feeds: NWS, SPC, FAA, NEXRAD, and NPS cover the United
  States; aircraft, weather, and space-weather feeds are global.
- Units are imperial (mi/ft, °F, mph); aircraft popups use aviation units
  (ft, kt). The UI is all-caps, units included — only a unit whose meaning
  depends on its case (SI symbols, a mixed-case index) would stay verbatim.
- An aircraft that breaches the range **and** the 1,000 ft AGL warning
  altitude flashes red/white on the map (a pulsing halo), in the SITREP
  title, and on its LOW AIRCRAFT card. Tapping the "Xs to update" badge
  forces an immediate refresh.
- AGL uses the ground elevation at the crosshair (cached per ~1 km cell) —
  a good approximation at ring radii; failed lookups are retried, never
  cached, so bad data can't suppress the traffic warnings.
- Every feed is keyless with `ACAO:*`; anything requiring a key is excluded
  by the keyless rule. One dead feed never breaks the board.
