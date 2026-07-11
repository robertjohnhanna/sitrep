# seehear

**One map. Every keyless public signal. Three audio domains you can click and hear.**

A single self-contained HTML file OSINT dashboard: live earthquakes, military
aircraft, emergency squawks, the ISS, satellites, natural disasters, severe
weather, aurora, geopolitics — plus a scanner-style radio panel spanning
**police/fire (OpenMHz)**, **shortwave/ham (KiwiSDR)**, and **aviation ATC
(LiveATC)** with live METARs. No other single tool covers all three audio
domains in one place.

No backend. No build step. No API keys. No `npm install`. **Open `index.html`
directly in a browser.** Fully responsive — works on iPhone, iPad, and desktop.

## Run it

```
open index.html      # or just double-click it
```

MapLibre GL and satellite.js load from CDN; every data feed is fetched
client-side from keyless public APIs. Nothing is installed or served.

## SITREP — situational awareness at a glance

On load, seehear locates you (instant IP fix via ipwho.is, refined by GPS if
you allow the browser prompt — denial is silently tolerated), flies the map to
your position, drops a pulsing marker, and builds a live **SITREP briefing**
that refreshes every 60 s:

The SITREP reflects the **current map area** (viewport), not a fixed point —
pan or zoom and it re-reads what's in view (nearest ATC tower, KiwiSDR,
aircraft, ships, quakes, disasters…). It has three heights: minimise (▾),
normal, and **▲ expand** to fill its whole bay over the map.

- **Anomalies** — a correlation pass across every in-view feed surfaces the
  outliers at the top (aircraft squawking 7500/7600/7700, red/orange GDACS
  alerts, M5+ quakes, plus global geomagnetic storms / M-X solar flares).
  When anything is flagged, the **SITREP title turns red**.
- ⚠ active weather alerts covering your exact point (NWS point query)
- 🌦 local conditions (Open-Meteo)
- ✈ every aircraft within 100 nm of you (auto-enabled layer, adsb.lol)
- 🎖 military aircraft within 300 km, with closest callsign
- 🚨 aircraft squawking 7700/7600/7500 worldwide, with distance to nearest
- 🌍 nearest earthquake and 🔥 nearest natural event, with distances
- 📟 **auto-tuned police/fire audio** — the OpenMHz system matching your
  city/region starts monitoring automatically
- 🛬 nearest ATC tower with a one-tap LISTEN chip
- 📻 nearest shortwave receiver with a one-tap OPEN chip
- 🌌 aurora-visibility hint when Kp ≥ 5 at your latitude

Every row has a VIEW/LISTEN/OPEN/TUNE action. The SITREP is a bottom bar
spanning the gap between the side panels (one-third of the viewport); its
header carries the space-weather badges (Kp, X-ray flare class), the CISA KEV
count, a UTC clock, your resolved location, and a **⌖ ME** button that
recenters on you. The map keeps its content centred in the open area above.
There is no top chrome — the two side panels (OBJECTS, left; radio, right) and
the SITREP bar are all anchored to the screen edges. Position is resolved
client-side and never leaves the browser except as anonymous lat/lon
parameters to the public weather APIs.

Units are imperial (mi/ft, °F, mph) and the interface is set in a minimalist
all-caps style, with measurement units left in natural case.

Every marker is a purpose-drawn icon: aircraft and ships are silhouettes
rotated to their heading/course, and hazards carry type-specific glyphs
(fire, volcano, cyclone, flood, launch, antenna, camera…). The map itself is
chrome-free — no on-map control widgets — with zoom by scroll/pinch, recenter
via ⌖ ME, and attribution in the panel footer. The two side panels and the
SITREP bar are all anchored to the bottom edge.

## Live map layers

| Group | Layer | Source | Refresh |
|---|---|---|---|
| Movement | Earthquakes (24 h, M2.5+) | USGS | 60 s |
| | Military aircraft | adsb.lol ADS-B | 60 s |
| | Aircraft near me (100 nm radius) | adsb.lol point query | 45 s |
| | Emergency squawks (7700/7600/7500) | adsb.lol | 60 s |
| | ISS live position | wheretheiss.at | 15 s |
| | Bright satellites + GPS constellation | CelesTrak TLE → SGP4 propagated in-browser | 60–120 s |
| | Amtrak trains | amtraker v3 | 60 s |
| | Ships / AIS (hull icon, heading-aligned) | Digitraffic (Baltic, keyless) | 60 s |
| | Starlink constellation (sampled) | CelesTrak TLE → SGP4 | 2 min |
| Geopolitics | Internet outages (by country) | IODA / Georgia Tech | 30 min |
| | Live cameras (curated public 24/7) | click-to-open | static |
| Hazard | Natural events — wildfires, storms, volcanoes, ice | NASA EONET v3 | 10 min |
| | Global disasters — quakes/cyclones/floods (alert-graded) | GDACS (UN/EC) | 30 min |
| | Rocket launches (upcoming, at the pad) | Launch Library 2 | 60 min |
| | Weather radar overlay | RainViewer | 5 min |
| | Live seismic (real-time, global) | EMSC seismicportal | 2 min |
| | River / flood gauges (in view) | USGS water services | 10 min |
| | Notable places (in view) | Wikipedia geosearch | on pan |
| | Severe-weather alert polygons | NOAA/NWS | 2 min |
| | Aurora forecast (≥40 % probability) | NOAA SWPC OVATION | 5 min |
| | Holocene volcanoes | Smithsonian GVP (curated static) | — |
| Geopolitics | Ukraine frontline | DeepStateMap | 30 min |
| | Conflict/protest events | GDELT 2.0 geo | 6 h |
| | Malware C2 count | abuse.ch Feodo | 5 min |

**Space-weather / cyber:** planetary Kp index, GOES X-ray flare class (NOAA
SWPC) and the CISA KEV catalog size ride in the SITREP conditions row; a UTC
clock sits in the SITREP header.

**Starts checked:** all aircraft (civil + military), NWS alerts, and weather
radar — other intel feeds load in the background so the SITREP and anomaly
detection stay complete without cluttering the map.

**Dossier:** right-click (desktop) or long-press (touch) anywhere →
reverse-geocoded place (Nominatim) + current weather (Open-Meteo) + country
profile (RestCountries) + head of state (Wikidata SPARQL) + Wikipedia summary.

**Search:** place name or raw `lat,lon` via OSM Nominatim.

## The radio panel — the point of this tool

1. **Police / Fire — OpenMHz.** Pick any of ~440 trunked-radio systems → live
   calls auto-play newest-last through one `<audio>` element, with a scan mode
   that cycles systems. Unencrypted talkgroups only; expect the
   trunk-recorder delay.
2. **Shortwave / Ham — full in-page KiwiSDR tuner.** Connects over the KiwiSDR
   WebSocket protocol, decodes its IMA-ADPCM audio, and plays via Web Audio —
   no plugin, no new tab. The **entire public receiver directory** (~600 nodes)
   loads from linkfanel's global-assigning JS file (no CORS needed), falling
   back to a curated list. Frequency entry, AM/LSB/USB/CW/FM, 13 band presets,
   live S-meter. **◎ NEAREST** tunes the closest receiver to your location;
   **SCAN** band-scans the HF spectrum. Nodes are http/ws, so run seehear from
   a local file. On an **https** deployment the browser blocks the
   mixed-content `ws://`, so tapping a receiver opens its native web tuner in
   a new tab instead; the in-page tuner also auto-falls-back to the native
   tuner if a node sends no audio within 8 s — so a receiver tap always
   produces sound somewhere.
3. **Aviation ATC — LiveATC + METARs.** 20 major world airports with live
   METAR weather (aviationweather.gov) inline in each row. ▶ tries the direct
   mp3 stream (`<audio>` playback isn't CORS-gated); ↗ opens LiveATC's
   official player as the guaranteed fallback.

## Mobile

Designed for iPhone/iPad as first-class targets: bottom MAP / LAYERS / RADIO
navigation with slide-up sheets, 44 px touch targets, safe-area (notch/home
indicator) insets, long-press dossier, `viewport-fit=cover`, momentum
scrolling. Desktop gets side-by-side panels.

## Design notes & honest caveats

- **`<audio>` playback is not gated by CORS** — only JSON `fetch()` is. So
  OpenMHz `.m4a` calls and LiveATC mp3 streams play cross-origin fine; only
  the OpenMHz *JSON API* carries a CORS/Cloudflare risk.
- **OpenMHz's JSON API sits behind a Cloudflare challenge.** It serves fine
  from a normal residential browser but may be challenged from datacenter/VPN
  IPs — if the systems list stays empty, that's why. Audio is unaffected.
- **Feeds without CORS headers** (abuse.ch, CISA KEV, GDELT, aviationweather)
  are kept with graceful degradation and a visible ⚠ marker — never silently
  dropped.
- **Dropped because they require a key** (violating the keyless rule):
  OpenAQ v3, NASA FIRMS (MAP_KEY), APRS.fi, OpenSky, aisstream, Shodan,
  Global Fishing Watch, Sentinel Hub.
- **Dropped because there's no keyless in-browser path:** raw APRS-IS (TCP)
  and Meshtastic MQTT (broker creds + protobuf).
- Every layer loader is defensive: one dead feed never breaks the board.

## Provenance

The list of public endpoints (and the keyless/keyed split) draws on
ShadowBroker's data-source table
([github.com/BigBodyCobain/Shadowbroker](https://github.com/BigBodyCobain/Shadowbroker),
AGPL-3.0). Only the *list of public endpoints* — public knowledge — is reused,
not any of their code. ShadowBroker is the full-fidelity Docker build; seehear
is deliberately the opposite: minimal, keyless, one file, three audio domains.
