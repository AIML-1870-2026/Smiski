# WeatherPal 2000™ — Deployment Spec

## Project Overview

A retro 2000s-era weather dashboard built as a single-file HTML app. It uses the OpenWeatherMap API to fetch live weather data and renders it in a Windows XP-inspired UI with animated radar, a scrolling ticker, and a 5-day forecast.

---

## File Structure

Deploy as a single static file — no build step required.

```
weatherpal-2000/
├── index.html        # Rename weather-app.html → index.html
└── README.md         # Optional
```

---

## Environment & Configuration

The API key is currently hardcoded in `index.html`:

```js
const API_KEY = '126bb9a062fd10331cf13c6ce27bbffe';
```

This is fine for a static GitHub Pages deploy, but if you want to keep the key out of your public repo, you have two options:

- **Option A (simple):** Keep it hardcoded. OpenWeatherMap free-tier keys have no sensitive billing risk.
- **Option B (safer):** Move to a backend proxy or use GitHub Secrets if you add a build pipeline later.

---

## Deployment Steps

### 1. Create the GitHub Repo

```bash
git init weatherpal-2000
cd weatherpal-2000
```

Rename your file and add it:

```bash
cp weather-app.html index.html
git add index.html
git commit -m "Initial commit: WeatherPal 2000"
```

### 2. Push to GitHub

```bash
gh repo create weatherpal-2000 --public --source=. --remote=origin --push
```

Or manually:

```bash
git remote add origin https://github.com/YOUR_USERNAME/weatherpal-2000.git
git branch -M main
git push -u origin main
```

### 3. Enable GitHub Pages

In your repo on GitHub:

1. Go to **Settings → Pages**
2. Under **Source**, select **Deploy from a branch**
3. Choose **main** branch, **/ (root)** folder
4. Click **Save**

Your app will be live at:

```
https://YOUR_USERNAME.github.io/weatherpal-2000/
```

(Takes ~1–2 minutes to propagate on first deploy.)

---

## API Details

| Property | Value |
|---|---|
| Provider | OpenWeatherMap |
| Endpoints used | `/data/2.5/weather`, `/data/2.5/forecast`, `/data/2.5/onecall` |
| Units | Imperial (°F) |
| Default city | Omaha, US |
| City query format | `{city},US` (state abbreviations are stripped automatically) |

---

## Known Limitations

- **CORS:** The app makes direct browser-side API calls. This works fine in a browser but will fail in sandboxed environments (like Claude's file preview).
- **Radar panel** is purely decorative — a CSS animation mimicking a radar screen (spinning sweep, pulsing rings, blinking dots). It is not connected to any real data. Live radar would require a paid OWM map tile layer or a third-party service like the Iowa Environmental Mesonet.
- **Rain % in forecast** is randomly generated for visual effect; real precipitation probability requires a paid OWM plan.
- No HTTPS enforcement needed — GitHub Pages serves over HTTPS by default.
- **Weather Alerts** require the `/onecall` endpoint, which may not be available on all free-tier API keys. If unavailable, the panel shows "No active alerts" gracefully.
- **UV Index** is sourced from `/onecall` using coordinates returned by the `/weather` endpoint. If the One Call API is unavailable, the UV panel shows N/A.

---

## Stretch Enhancements

Three features added beyond the base dashboard:

### 1. °C / °F Toggle
A button in the toolbar switches all temperature displays between Fahrenheit and Celsius in real time — including the main temp, feels-like, hourly strip, and 5-day forecast. No re-fetch required; conversion is handled client-side via `displayTemp()`.

### 2. UV Index Panel
Uses the `/data/2.5/onecall` endpoint (fetched after the initial weather call using the city's lat/lon). Displays:
- Numeric UV index value
- Category label (Low / Moderate / High / Very High / Extreme) with color-coded badge
- Gradient meter bar with a needle indicating current UV position

| UV Index | Category | Color |
|---|---|---|
| 0–2 | Low | Green |
| 3–5 | Moderate | Yellow |
| 6–7 | High | Orange |
| 8–10 | Very High | Red |
| 11+ | Extreme | Purple |

### 3. Weather Alerts Panel
Also sourced from `/data/2.5/onecall`. Displays up to 3 active NWS alerts with event name, start/end times, and issuing agency. Shows a blinking yellow indicator when alerts are active. Falls back gracefully to a "No active alerts" message when the area is clear.


---

## Future Improvements (optional)

- Add `geolocation` API support to auto-detect user's city on load
- Replace simulated rain % with real `pop` field from OWM forecast response
- ~~Add °C / °F toggle~~ ✅ Implemented
- Store last searched city in `localStorage`
- Hook up a real radar tile layer via Leaflet + OWM map tiles
