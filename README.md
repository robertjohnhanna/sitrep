# canifly

**Can I fly my drone here, now, and how high? One keyless, single-file map
that answers it for wherever the crosshair points.**

A self-contained HTML app: a dark map with a centre crosshair and a docked
panel (a bottom sheet on mobile). The panel leads with the **flyability
chart** — can a small drone fly *here, now, and for the next three hours, and
how high* — plus a max-wind setting (and a range stepper on the map) and a **SITREP** briefing: a
stack of distance-sorted cards for everything inside the range ring — low
manned traffic, FAA airspace ceilings, NWS warnings, park no-fly land,
severe-weather outlooks, space weather, and the local forecast.

No backend. No build step. No API keys. No `npm install`. **Open `index.html`
directly in a browser.** Fully responsive — iPhone, iPad, and desktop.

## Run it

```
open index.html      # or just double-click it
```

MapLibre GL loads from CDN; every data feed is fetched client-side from
keyless public APIs.

## The flyability chart

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

The bold altitude label marks the highest currently flyable row. **Max wind**
(±1 mph) is set in the **settings box** below the chart; **range** (±0.5 mi,
0.5–15) has its own stepper in the floating control at the **bottom-right of the
map** — `(−) RANGE ⌖ n.n MI (+)`, which also holds the center-on-me (⌖) button.
Both settings are persisted per device. Range drives the FAA query radius, the
traffic/radar sweep, the SITREP card radius, and the dashed **range ring** drawn
around the crosshair. Zoom is capped so the ring never exceeds ⅓ of the view.

## The SITREP cards

Everything is measured from the **crosshair** (map centre) and limited to the
range ring. Cards are tiered — red hazards, amber hazards, then routine
traffic — and distance-sorted within each tier (capped at 14, plus pinned and
context cards):

- 🛩️ **LOW AIRCRAFT** — when a plane is in range **and** under 1,000 ft AGL
  (AGL = QNH-corrected altitude − ground elevation under that plane; see below),
  that plane's own card transforms into the red flashing LOW AIRCRAFT alert and
  sorts to the top; it reverts to the normal ✈️/🪖/🚨 card the moment it
  climbs out or leaves. The card shows the callsign and **ground speed** plus its
  AGL and bearing. Always exactly one card per plane. The same deduped collector
  feeds the chart's TRFC cells, so card, chart and map halo agree.
- ⛔ **FAA NO-FLY / FAA CEILING** — surfaces the chart's own airspace gate: it
  reads the exact `aspCapFt` the flyability chart's FAA column computes (the
  FAA LAANC UASFM grid + defense TFRs swept over the range ring), so the card
  and the chart can never disagree. Red "no-fly" when the LAANC ceiling in
  range is 0, amber "ceiling N ft" when it caps below 400. Appears whenever the
  chart is FAA-limited — even where no charted-airspace polygon is in range
  (the LAANC grid's 0-ft cells follow approach corridors, not the drawn
  bubbles).
- 🚨 emergency squawks (7700/7600/7500) · ⚠️ NWS **warnings** (not watches) ·
  ⛔/⚠️ FAA restrictions (defense, prohibited, security, MOA, stadium…) ·
  🏞️ NPS land · 🗼 controlled airspace (Class B/C/D/E) · ✈️ / 🪖 civil and
  military traffic. Every airspace card is tinted to match its **map polygon**
  colour — blue Class B/D, magenta Class C/E, red defense/prohibited, orange
  restricted, gold MOA/warning/stadium — so a card reads at a glance as the
  zone drawn on the map. (Controlled-airspace tints are context — authorization
  required — and don't drive the title; the real ceiling severity stays on the
  FAA card above.)
- Context cards at the bottom: 🌪️/⛈️ SPC Day-1 outlooks, 🧲 geomagnetic
  storms (Kp ≥ 5), ☀️ M/X solar flares, and the 🌡️ local forecast (always
  present).

Tap any card to fly the map there (airspace/TFR zones recentre on the polygon
centre; other polygons fly to their nearest edge — the same edge distance used
for sorting and range inclusion). The **SITREP title**
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
| NEXRAD reflectivity mosaic | Iowa State Mesonet | 30 s |
| SPC Day-1 outlook (hidden; feeds cards) | NOAA SPC | 15 min |
| FAA airspace + restrictions (regional cache) | FAA ArcGIS ×5 services | on travel |
| NPS lands (regional cache) | NPS ArcGIS | on travel |

**Two refresh classes.** *Dynamic* feeds (aircraft, radar, weather, Kp, NWS,
SPC) ride the 30 s pulse and refetch on pan/zoom. *Static* feeds (FAA airspace,
NPS lands) barely change, and the full US is ~50 MB (times out) — so they're a
**regional cache**: pulled once for a ~85-mile box around you, drawn at *every*
zoom, and re-pulled only when you travel out of that region. That's why
airspace stays painted when you zoom out, and it never touches the periodic
pulse. Every loader retries with backoff and a failure never wipes the map.

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
- AGL is computed per plane: **QNH-corrected** barometric altitude (ADS-B
  reports pressure altitude off 29.92″; the local sea-level pressure from the
  weather feed corrects it, ~27 ft/hPa) minus the **ground elevation under that
  plane** (Open-Meteo, cached per ~1 km cell, batched into one request, with the
  crosshair cell as fallback). Failed elevation lookups are retried, never
  cached, so bad data can't suppress the traffic warnings. Geometric-only
  altitudes (no baro) are used as-is (already ≈ MSL).
- Every feed is keyless with `ACAO:*`; anything requiring a key is excluded
  by the keyless rule. One dead feed never breaks the board.
