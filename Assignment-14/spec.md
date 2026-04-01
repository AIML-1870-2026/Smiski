# NEO Watch Dashboard — Spec

## Overview

A single-file, client-side HTML dashboard that pulls live NASA data and visualizes near-Earth object (NEO) activity for the current 7-day window. No build step or server required — open the file in a browser and it works. A fixed NASA API key (`LOWvtLzuGPGdRRvsSv5BKCBq71qpDTmAaq0QHKaS`) is embedded in the JS.

---

## Visual Design System

**Theme:** Deep-space dark with galaxy purple tones.

**CSS Variables (`:root`):**
| Variable | Value | Role |
|---|---|---|
| `--bg` | `#0a0520` | Page background |
| `--surface` | `#130a30` | Panel / card surface |
| `--border` | `rgba(180,120,255,0.25)` | Subtle purple border |
| `--accent` | `#c87eff` | Primary purple accent |
| `--accent2` | `#ffe566` | Yellow / gold highlight |
| `--accent3` | `#66e0ff` | Cyan highlight |
| `--warn` | `#ffb347` | Warning orange |
| `--danger` | `#ff6b8a` | Danger red-pink |
| `--text` | `#d4c0f8` | Body text |
| `--muted` | `#7060a0` | Subdued / label text |
| `--head` | `#ffffff` | Headings / prominent values |

**Fonts:**
- `Fredoka One` (Google Fonts) — display headings, section headers, tab labels, large metric values
- `Nunito` (Google Fonts) — all body text, labels, table cells

**Backgrounds:**
- Page: `#04020f` with two radial-gradient overlays (purple ambient glows) + fixed twinkling star canvas
- Root container: `linear-gradient(160deg, #0d0528, #160840, #0d052a)` with purple border and box-shadow glow
- Header: `linear-gradient(135deg, #1a0840, #2d1060, #1a0840)` with an infinite horizontal shimmer animation

**Animations:**
- `shimmer` — horizontal sweep on header
- `pulse` — live-data pulsing dot
- `spin` — loading spinner
- `luma-float` — Luma pet floating bob
- `sparkle-fly` — click sparkle particles
- `star-spin` — rotating star badge in header
- Star canvas: 220 randomly placed stars with individual twinkle (alpha oscillation) via `requestAnimationFrame`

---

## Layout Structure

```
<body>
  <canvas id="star-canvas">        ← fixed, full-viewport star field
  #luma-pet                        ← fixed bottom-right mascot
  #luma-bubble                     ← speech bubble for mascot

  #root (max-width: 1140px)
    .hdr                           ← header bar
    .tabs                          ← tab strip (5 tabs)
    #tab-week .panel               ← This Week
    #tab-globe .panel              ← 3D Globe
    #tab-approaches .panel         ← Close Approaches
    #tab-hazardous .panel          ← Hazardous NEOs
    #tab-apod .panel               ← Space Media
```

---

## Header (`.hdr`)

- Rotating star badge emoji (`⭐`) with `star-spin` animation
- Logo line: `NASA / JPL` (small, yellow, letter-spaced)
- Title: `NEO WATCH DASHBOARD` (Fredoka One, white, purple glow)
- Right side: pulsing cyan dot + `LIVE DATA` label

---

## Tab System

Five tabs rendered as `<button class="tab">` elements inside `.tabs`. Active tab highlighted in `--accent2` (yellow) with a bottom border. Tab switching is handled by `switchTab(id)` which:
1. Toggles `.active` on `.tab` buttons
2. Toggles `.active` on `.panel` divs
3. Lazily initializes the 3D globe on first visit to that tab (if data is loaded)

| Tab ID | Label | Icon |
|---|---|---|
| `week` | This Week | ⭐ |
| `globe` | 3D Globe | 🌍 |
| `approaches` | Close Approaches | ☄️ |
| `hazardous` | Hazardous NEOs | ⚠️ |
| `apod` | Space Media | 🔭 |

---

## Data Sources & API Calls

### NASA NeoWs (Near Earth Object Web Service)
- **Endpoint:** `https://api.nasa.gov/neo/rest/v1/feed`
- **Params:** `start_date` = today, `end_date` = today + 7 days, `api_key`
- **Loaded by:** `loadNeoWs()` on boot
- **Stored in:** `neoData` — flat array of `{ obj, ca, date }` sorted by date
- **Populates:** This Week, Close Approaches, Hazardous NEOs tabs; fallback data for 3D Globe

### 3D Globe Data Source
- The 3D Globe tab uses `neoData` (already loaded by `loadNeoWs()`) as its data source
- No additional API call is made inside `initGlobe()` — avoids CORS issues that blocked the JPL CAD API from GitHub Pages
- PHA flags come directly from `obj.is_potentially_hazardous_asteroid` in the NeoWs response
- All asteroids in the 7-day window are plotted (no limit)

### NASA APOD (Astronomy Picture of the Day)
- **Endpoint:** `https://api.nasa.gov/planetary/apod`
- **Params:** `count=4`, `api_key`
- **Loaded by:** `loadAPOD()` on boot
- **Populates:** Space Media tab (1 featured + 3 gallery items)

---

## Tab: This Week (`#tab-week`)

Rendered by `renderWeek()`. Requires `neoData` to be populated.

**Hero grid (3-column, responsive):** 6 stat cards
1. Total Objects — total entries in `neoData`
2. Potentially Hazardous — count of PHAs
3. Closest Approach — min miss distance, name, lunar-distance context
4. Fastest Flyby — max velocity in km/h, ISS speed multiplier
5. Largest Object — estimated diameter in meters
6. Data Window — today's date, source label

**Velocity Context section:** 4 metric cards showing how many times faster the fastest asteroid is than: speed of sound (1,235 km/h), rifle bullet (3,960 km/h), ISS orbital speed (27,600 km/h), Earth escape velocity (40,320 km/h).

**Size Comparison section:** Visual bubble chart comparing familiar objects (car 4.5m, bus 12m, Boeing 747 64m, Eiffel Tower 330m, Empire State 443m) to the closest asteroid's estimated diameter. Bubbles are circles sized proportionally, with the asteroid highlighted in purple.

---

## Tab: 3D Globe (`#tab-globe`)

Rendered by `initGlobe()` — called lazily when the tab is first activated.

**Libraries:**
- `globe.gl` v2.31.0 (via unpkg CDN)
- `three.js` r128 (via cdnjs CDN)

**Globe setup:**
- Earth texture: `three-globe` blue marble image
- Background: `three-globe` night sky image
- Container: `#globe-wrap`, `560px` tall, dark background
- Auto-rotates at speed 0.35; zoom enabled; initial altitude 3.5

**Object rendering:**
- All asteroids from `neoData` (the full 7-day NeoWs feed, typically 50–100 objects)
- Each asteroid assigned a random lat/lng and an altitude derived from `distToAlt(km)` — a log-scale function compressing 0.01–1000 LD into globe.gl altitude units 0.3–3.8
- Positions set via `myGlobe.getCoords(lat, lng, alt)` in `customThreeObjectUpdate` — uses globe.gl's own coordinate transform
- Non-hazardous asteroids: teal spheres (`0x00c8b4`), radius 3.5
- Potentially hazardous asteroids (PHAs): red spheres (`0xe84040`), radius 5.5; detected via separate `pha=true` CAD API request
- Moon: light purple/blue sphere (`0xd8d8ff`), radius 10, placed at `alt = distToAlt(384400 km)` = 1 LD

**Moon orbit:**
- Orbits at `MOON_ORBIT_SPEED = 0.003` rad/frame via `requestAnimationFrame` inside `onGlobeReady`
- Equatorial orbit (lat = 0), longitude increments each frame; position computed via `myGlobe.getCoords(0, lng, MOON_ALT)`
- THREE.js group position updated directly each frame (bypasses reactive data layer)
- A visual ring at the Moon's orbital radius, computed as `(MOON_ALT + 1) * 100` scene units (semi-transparent blue-purple, `RingGeometry`)
- A second inner reference ring at scene unit 130 (roughly LEO reference, teal, very faint)

**Lighting:**
- `AmbientLight` at `0x888888`
- `DirectionalLight` at `0xffffff`, intensity 1.1, position (400, 300, 200)

**Legend:** Three items below the globe — non-hazardous, PHA, Moon.

**Detail panel (`#asteroid-detail`):** Clicking any asteroid or the Moon populates this panel with a key-value table. Fields shown:
- Asteroid: Object name, Close Approach date, Miss Distance (LD context), Miss Distance (km), Intuitive distance (lunar trips around Earth), Velocity (km/h + ISS multiplier), Estimated Diameter, Hazardous flag
- Moon: fixed reference data (orbital period, speed, LD definition)

**Status line (`#globe-status`):** Shows asteroid count, scaling note, interaction hint after globe ready.

---

## Tab: Close Approaches (`#tab-approaches`)

Rendered by `renderApproaches()`.

**Metrics row (`.mrow`):** 4 stat cards — Total Objects, Hazardous count, Closest miss distance, Source label.

**Filter controls:**
- Sort by dropdown: Date | Miss Distance | Velocity | Diameter
- "Hazardous Only" checkbox

**Table** (up to 60 rows): Object name, Date, Miss Dist, Lunar Dist (LD), Velocity, Est. Diam, Status tag.

Status tags are inline `<span>` elements:
- `.td` — red/pink — `HAZARDOUS`
- `.tok` — cyan — `SAFE`

---

## Tab: Hazardous NEOs (`#tab-hazardous`)

Rendered by `renderHazardous()`. Shows only PHAs.

**Definition note:** PHAs defined as NEOs with H < 22 and miss distance < 0.05 AU.

**Cards grid** (auto-fit, min 320px): One card per PHA with:
- Name, date, PHA badge
- Miss Distance (formatted + LD)
- Velocity (km/h + ISS multiplier)
- Diameter bar (proportional to 2,000m max, red fill)
- Velocity bar (proportional to 200,000 km/h max, orange fill)
- Size context label: House-sized / City-block-sized / Skyscraper-sized / Mountain-sized / City-killer class

If no PHAs this week, shows a "clear sky" message in accent color.

---

## Tab: Space Media (`#tab-apod`)

Rendered by `loadAPOD()`.

**Two-column layout** (responsive, collapses to 1 col below 680px):
- Left: Featured APOD — full image (`<img>` with HD URL fallback) or video link, title, date, copyright, full explanation text
- Right: "Recent Gallery" — 3 mini-cards with thumbnail (82×62px), title, date, truncated explanation (90 chars)

Handles both `image` and `video` media types.

---

## Utility Functions

| Function | Purpose |
|---|---|
| `fmt(d)` | Formats a Date to `YYYY-MM-DD` |
| `fmtDist(km)` | Smart distance: km if < 0.1 LD, LD if < 10 LD, else millions km |
| `fmtDistVerbose(km)` | Returns `X.XX LD · ~N trips around Earth` |
| `estDiamM(obj)` | Average of min/max estimated diameter in meters |
| `hazTag(bool)` | Returns HTML for HAZARDOUS or SAFE status tag |
| `distToAlt(km)` | Log-scales km to globe.gl altitude units (0.3–3.8); internally maps to scene units 130–480 |

---

## Luma Pet Mascot

Fixed bottom-right (`bottom: 28px, right: 28px`), 56×56px SVG of a golden star character with two eyes and rosy cheeks. Always visible, floating animation (`luma-float`, 3s).

**Interaction:**
- Click triggers `lumaClick()`: shows a random message in `#luma-bubble`, spawns 10 radial sparkle particles that fly outward and fade
- Auto-nudge: every 14 seconds, 25% chance of auto-triggering `lumaClick()`
- 15 possible messages ranging from asteroid jokes to general space enthusiasm

**Bubble:** Fixed above the pet, fades in/out with CSS transition, disappears after 2.8 seconds.

---

## Reusable Component Patterns

### Metric Card (`.mc`)
```html
<div class="mc">
  <div class="mc-lbl">LABEL</div>
  <div class="mc-val [warn|sm]">VALUE</div>
  <div class="mc-sub">subtext</div>
</div>
```

### Section Header (`.sh`)
```html
<div class="sh">Section Title</div>
```
Yellow, Fredoka One, bottom border, subtle glow.

### Progress Bar
```html
<div class="rbar-wrap">
  <div class="rbar-bg"><div class="rbar-fill" style="width:X%;background:COLOR"></div></div>
  <span class="rval">label</span>
</div>
```

### Loading / Error States
- Loading: `.loading` with `.spin` spinner and message
- Error: `.err` with danger color

---

## Boot Sequence

On page load (bottom of `<script>`):
1. `loadNeoWs()` — fetches NeoWs, on success calls `renderWeek()`, `renderApproaches()`, `renderHazardous()`, and `initGlobe()` if globe tab is already active
2. `loadAPOD()` — fetches APOD independently
3. Star canvas IIFE starts immediately

Globe initialization (`initGlobe()`) is deferred until first visit to the 3D Globe tab, guarded by `globeReady` boolean flag.
