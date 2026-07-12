# canifly

**Can I fly my drone here, now, and how high? One keyless, single-file map
that answers it for wherever the crosshair points.**

A self-contained HTML app: a dark map with a centre crosshair and a docked
panel (on mobile, two full-screen pages — the map and the panel — that you flip
between with a single tap). The panel leads with the
**flyability chart** — can a small drone fly *here, now, and for the next three
hours, and how high* — plus a max-wind setting (and a range stepper on the map)
and a **SITREP** briefing: a stack of distance-sorted cards for everything
inside the range ring — low manned traffic, FAA airspace ceilings, NWS warnings,
park no-fly land, severe-weather outlooks, space weather, and the local forecast.

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
| KPx | planetary Kp ≥ 7 (G3, GPS degraded) grounds; Kp 5–6 (G1–G2) is an amber caution. NOW uses the finalized observed bin; +1/+2/+3h use the forecast | NOAA SWPC |
| FAA | LAANC grid ceiling caps the band; a National-Defense TFR in range grounds | FAA ArcGIS (point + range buffer) |
| PROH / NSUF / PARK | Prohibited area, security UAS zone, or NPS land under the crosshair grounds | FAA ArcGIS · NPS |
| PCPN | NEXRAD echo inside the range ring grounds NOW | Iowa State Mesonet mosaic (pixel-sampled) |
| TRFC | manned aircraft within the low-aircraft net (3× the ring) caps the NOW column: the drone must stay 500 ft below it (§91.119 min altitude), so it flashes red/white at that ceiling and reds every row above | ADS-B |

**Fail-safe posture:** a feed that can't be verified never silently reads
"clear". Unknown inputs fail *open* for the grid values but amber the NOW
header (hover for which inputs are unverified) and the SITREP title. The FAA
airspace query hard-throws on anything that could understate a restriction
(wrong units, truncated results, missing ceilings). When a gate input stays
unverified past a short grace window, a **📡 DATA card** spells out which feeds
are stale (see the SITREP cards) — amber while the chart still renders from
last-good data, red when there's no verdict at all.

The bold altitude label marks the highest currently flyable row. **Max wind**
(±1 mph) is set in the **settings box** below the chart; **range** (±0.5 mi,
0.5–15) has its own stepper in the floating control at the **bottom-centre of the
map** — `(−) RANGE ⌖ n.n MI (+)`, which also holds the center-on-me (⌖) button.
Both settings are persisted per device. Range drives the FAA query radius, the
radar sweep, the SITREP card radius, and the dashed **range ring** drawn around
the crosshair. A second, **light-grey dotted ring at 3× the range** marks the
low-aircraft warning net (see LOW AIRCRAFT below) — the same 3× radius the zoom
cap uses, so one knob keeps them in lock-step. Zoom is capped so the range ring
never exceeds ⅓ of the view — and the cap tracks the ring **both ways**: shrink
the range and you can zoom in further; widen it and the camera eases back out.
Both the range ring and the range control's own outline are tinted to the
current verdict — green clear, amber reduced/unverified, red grounded — so the
one-glance colour reaches the map even when the panel is off-screen.

## The SITREP cards

Everything is measured from the **crosshair** (map centre) and limited to the
range ring. Cards are tiered — red hazards, amber hazards, then routine
traffic — and distance-sorted within each tier (capped at 14, plus pinned and
context cards):

- ✈️/🚁 **LOW AIRCRAFT** — when a plane is under the warning altitude **and**
  inside the low-aircraft warning net, its card transforms into the red flashing
  LOW AIRCRAFT alert and sorts to the top; it reverts the moment it climbs out or
  leaves. The warning altitude is the **400 ft drone ceiling + 500 ft separation
  = 900 ft AGL** (AGL = QNH-corrected altitude − ground elevation under that
  plane; see below): a plane below that pushes its required-clearance floor into
  the drone band. Part 107 sets no *numeric* drone-vs-aircraft separation — the
  rule is simply "yield the right of way" (§107.37) — so 500 ft is used as the
  buffer, the manned minimum safe altitude over non-congested areas (§91.119(c)),
  i.e. the closest a plane should legally get to the 400 ft band. The net itself
  is **3× the range ring** (the light-grey dotted ring), a wider reach than the
  rest of the SITREP so low traffic is flagged early. (A breaching plane beyond
  the range ring but inside the grey ring gets a card too; ordinary cruising
  traffic out there does not.) Exactly one card per plane. The same collector +
  radius + separation feeds the chart's TRFC ceiling, so card, chart and map halo
  agree. The chart's **TRFC** cell marks the highest usable band (plane AGL − 500,
  clamped to the grid), flashing at that ceiling with every row above it red and
  the rows below green. Every plane card shows its ADS-B **ground speed** (MPH) on
  line 3, right after the altitude.
- ⛔ **FAA NO-FLY / FAA CEILING** — surfaces the chart's own airspace gate: it
  reads the exact `aspCapFt` the flyability chart's FAA column computes (the
  FAA LAANC UASFM grid + defense TFRs swept over the range ring), so the card
  and the chart can never disagree. Red "no-fly" when the LAANC ceiling in
  range is 0, amber "ceiling N ft" when it caps below 400. Appears whenever the
  chart is FAA-limited — even where no charted-airspace polygon is in range
  (the LAANC grid's 0-ft cells follow approach corridors, not the drawn
  bubbles).
- 🧲 **SPACE WEATHER** — a geomagnetic Kp storm degrades the GPS/compass a drone
  relies on, right-sized to actual impact: **red grounding at Kp ≥ 7** (G3, where
  NOAA flags GPS degradation), **amber caution at Kp 5–6** (G1–G2, accuracy may
  drop but flyable). It reads the same `kpNow` the chart's KP gate uses — the
  **finalized observed** value for NOW (the in-progress estimate is preliminary
  and jumpy), forecasts for the +1/+2/+3h columns — so card and chart agree.
- 📡 **DATA** — the flyability gates fail *open*, so an unverifiable feed keeps
  its last-good value and only ambers the NOW header quietly. If **FAA airspace,
  Kp, or radar** stays stale past a ~35 s grace window (a self-healing blip won't
  pop it), this amber card names them. Tap to force a refresh; it clears the
  moment every feed is fresh. (Weather health rides the weather card itself — see
  below — so there's never a duplicate card for the same feed.)
- 🌡️ **WEATHER** — there's one weather feed, so there's **one weather card**, and
  it changes rank, colour and text with its own severity (no separate transient
  "weather warning" duplicating the forecast):
  - a weather **gate** limiting the NOW column **promotes** it to a top hazard —
    precipitation, high gusts, winds-aloft-over-max and low visibility **ground**
    (red ⛈️/💨/🌫️); wind aloft or a low cloud base instead **cap the ceiling**
    (amber "wind/cloud ceiling N ft"). The headline is the gate and the measured
    value; the **current conditions fold onto line 3**, so the sky/temp/wind show
    exactly once. It reads the chart's own NOW gates, so card and chart agree.
  - otherwise it's the routine **conditions card at the bottom** (temp, sky,
    wind, feels-like, hi/lo), which also carries feed health: **amber**
    ("stale — tap") when the feed is failing but last-good conditions still show,
    **red** ("weather unavailable", promoted) when it never loaded and the chart
    can't compute — the red case reddens the title so a total outage can't hide
    behind a green skeleton.
- emergency squawks (7700/7600/7500) · ⚠️ NWS **warnings** (not watches) ·
  civil and military traffic. Every aircraft card uses just **two icons** — ✈️
  plane / 🚁 helicopter (from the ADS-B rotorcraft category) — for all of them,
  low-aircraft included. Which category it is (civil / military / emergency) is
  carried by **colour instead**: the tail-number string is tinted to match the
  plane/heli icon drawn on the map — **green** civil, **yellow** military, **red**
  emergency — on both the regular and the LOW AIRCRAFT card.
  Airspace cards follow a **solid three-way
  model** — *authorization* (controlled airspace, needs a ceiling) is not the
  same as a *restriction* (a stay-out zone), and neither is an *advisory*:
  - **hard restrictions** — 🛡️ defense, ⛔ prohibited, 🔒 security (NSUFR) — are
    red and **ground** the chart: drones are prohibited outright.
  - **conditional restrictions** — ⚠️ restricted areas and 🏟️ stadium TFRs — are
    amber "no-fly if active" / "no-fly during events". They're only active on a
    schedule the app can't read live, so per fail-safe posture they *warn* but
    don't silently ground a flight that may be perfectly legal.
  - 🎯 **advisory** — MOAs, warning / alert / danger areas — carry no colour and
    no title weight ("advisory only"): they concern manned military traffic, not
    a Part-107 drone in the 400-ft band, so treating them as restrictions would
    be a false alarm.

  (Each airspace card's icon is class-specific — 🛡️ defense · 🔒 security · ⛔
  prohibited · ⚠️ restricted · 🏟️ stadium · 🎯 military/special-use · 🗼 controlled.)
  - 🗼 **controlled airspace** (Class B/C/D/E) reads "authorization (LAANC)" —
    context that you need a ceiling here; the real ceiling severity stays on the
    FAA card above, and the tint (blue Class B/D, magenta Class C/E) doesn't
    drive the title.

  Every zone whose **floor is above 400 ft** is dropped entirely — it can't
  touch the drone band — so the cards only ever show airspace that actually
  matters at flight altitude. Each card is still tinted to match its **map
  polygon** colour (red defense/prohibited, orange restricted, gold advisory) so
  it reads at a glance as the zone drawn on the map.
- Context cards at the bottom: ☀️ M/X solar flares (an HF-blackout risk, but not
  a chart grounding gate), then 🌪️/⛈️ SPC Day-1 **outlooks** — *info only*, no
  colour and no title contribution (an outlook is "expected", not a warning) —
  sitting just above the 🌡️ local forecast (always present).

Tap any card to fly the map there (airspace/TFR zones recentre on the polygon
centre; other polygons fly to their nearest edge — the same edge distance used
for sorting and range inclusion). The **SITREP title**
is the one-glance verdict: red when the NOW column is grounded or a red hazard
is in range, amber for a reduced ceiling, an amber hazard, or any unverified
gate input.

## Map layers

| Layer | Source | Refresh |
|---|---|---|
| Aircraft (250 nm around centre, civil) | airplanes.live → adsb.fi fallback | on move · 15 s |
| Military aircraft (worldwide) | ADS-B `/mil` | 15 s |
| Emergency squawks (worldwide) | ADS-B `/squawk` | 15 s |
| NWS warning polygons (US) | api.weather.gov | 15 s |
| NEXRAD reflectivity mosaic | Iowa State Mesonet | 15 s |
| SPC Day-1 outlook (hidden; feeds cards) | NOAA SPC | 15 min |
| FAA airspace + restrictions (regional cache) | FAA ArcGIS ×5 services | on travel |
| NPS lands (regional cache) | NPS ArcGIS | on travel |

**Two refresh classes.** *Dynamic* feeds (aircraft, radar, weather, Kp, NWS,
SPC) ride the 15 s pulse and refetch on pan/zoom. *Static* feeds (FAA airspace,
NPS lands) barely change, and the full US is ~50 MB (times out) — so they're a
**regional cache**: pulled once for a ~85-mile box around you, drawn at *every*
zoom, and re-pulled only when you travel out of that region. That's why
airspace stays painted when you zoom out, and it never touches the periodic
pulse. Every loader retries with backoff and a failure never wipes the map.

The map has **no click popups** — everything inside the range ring is described
by the SITREP cards, so the map stays a clean picture and the panel carries the
detail.

**Mobile:** the map and the CANIFLY panel are two full-screen pages. **Tap the
map** to bring the panel up; **tap the flyability chart** (or any card) to drop
back to the map. No drag, no snap points — one tap flips the page, and each page
gets the whole screen. The panel uses the dynamic viewport height so it never
spills below Safari's address bar. This two-page toggle is **portrait-only** — a
phone in **landscape** is wide enough for the docked-right panel, so it falls
through to the same layout as a tablet or desktop.

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
- Units are imperial (mi/ft, °F, mph). The UI is all-caps, units included — only a unit whose meaning
  depends on its case (SI symbols, a mixed-case index) would stay verbatim.
- An aircraft that breaches the net **and** the 900 ft AGL warning altitude
  (400 ft ceiling + 500 ft separation) flashes red/white on the map (a pulsing
  halo), in the SITREP title, on its LOW AIRCRAFT card, and on the chart's
  **TRFC** ceiling cell (which only appears while a plane breaches, so all four
  flash in sync). Tapping the "Xs to update" badge forces an immediate refresh.
- AGL is computed per plane: **QNH-corrected** barometric altitude (ADS-B
  reports pressure altitude off 29.92″; the local sea-level pressure from the
  weather feed corrects it, ~27 ft/hPa) minus the **ground elevation under that
  plane** (Open-Meteo, cached per ~1 km cell, batched into one request, with the
  crosshair cell as fallback). Failed elevation lookups are retried, never
  cached, so bad data can't suppress the traffic warnings. Geometric-only
  altitudes (no baro) are used as-is (already ≈ MSL).
- Every feed is keyless with `ACAO:*`; anything requiring a key is excluded
  by the keyless rule. One dead feed never breaks the board.
