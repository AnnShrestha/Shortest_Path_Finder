# Shortest Route Finder

A fully client-side, single-file web application for finding the shortest route between two points on a map. Built with Leaflet.js and the OSRM routing engine — **no API key, no backend, no installation required.**

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [How to Use](#how-to-use)
- [User Interface](#user-interface)
- [Travel Modes](#travel-modes)
- [Road Blockages](#road-blockages)
- [Road Type Exclusions](#road-type-exclusions)
- [Address Search](#address-search)
- [GPS Location](#gps-location)
- [Map Styles](#map-styles)
- [Turn-by-Turn Directions](#turn-by-turn-directions)
- [Route Information Panel](#route-information-panel)
- [Technical Details](#technical-details)
- [Dependencies](#dependencies)
- [Limitations](#limitations)

---

## Overview

Shortest Route Finder lets you click two points on a world map and instantly see the fastest road route between them. You can simulate road blockages, exclude road types (motorways, tolls, ferries), switch between driving / cycling / walking modes, and get step-by-step turn-by-turn directions — all inside a single HTML file that runs entirely in your browser.

---

## Features

| Feature | Details |
|---|---|
| Click-to-route | Click start and end points directly on the map |
| Address search | Search by place name or address (both points) |
| GPS location | Use your device's current location as start or end |
| Travel modes | Driving, Cycling, Walking |
| Road blockages | Place, drag, and remove blockage markers; route automatically reroutes around them |
| Road exclusions | Exclude motorways, toll roads, and/or ferry routes |
| Map styles | CARTO Light, CARTO Dark, Satellite (Esri) |
| Turn-by-turn directions | Full step list with SVG maneuver icons and distances |
| Route stats | Total distance (km), estimated travel time, blockages avoided |
| No API key | Uses free public OSRM and Nominatim services |
| No installation | Open the HTML file directly in any modern browser |

---

## How to Use

1. **Open** `shortest-route-map.html` in any modern browser (Chrome, Firefox, Edge, Safari).
2. **Click the map** to set your start point (green **A** pin).
3. **Click the map again** to set your end point (red **B** pin).
4. The route is calculated and drawn automatically.
5. Use the sidebar controls to refine the route (mode, blockages, exclusions).

To start over, click the **Reset** button at the bottom of the sidebar.

---

## User Interface

The app is divided into two panels:

- **Sidebar (left, 300 px)** — all controls, search, options, and route results.
- **Map (right, full height)** — interactive Leaflet map with the route drawn on it.

The sidebar is scrollable. The map fills the remaining viewport width.

### Step Indicator

Three steps are shown at the bottom of the sidebar:

| Step | Meaning |
|---|---|
| **Start** | Start point has been set |
| **End** | End point has been set |
| **Route** | Route calculated — shows distance and duration |

Each step turns blue and shows a checkmark when complete.

---

## Travel Modes

Select a travel mode using the three buttons at the top of the sidebar:

| Button | Mode | OSRM Profile |
|---|---|---|
| Car icon | **Driving** | `driving` |
| Bike icon | **Cycling** | `cycling` |
| Walk icon | **Walking** | `foot` |

Switching mode while a route is displayed immediately recalculates the route. Different modes use different road networks and speed profiles, so results (distance, time, path) will differ.

---

## Road Blockages

Blockages simulate closed or blocked roads and force the router to find an alternative path.

### Placing a Blockage

1. Click **Block a Road Segment** in the *Road Conditions* section.
2. The button turns red and shows a pulsing indicator — blockage mode is active.
3. Click anywhere on the map near a road to place a blockage (red circle with X).
4. The route immediately recalculates to avoid the blockage.
5. Click the button again to exit blockage mode.

### Managing Blockages

| Action | How |
|---|---|
| Reposition | Drag the blockage marker to a new location |
| Remove one | Right-click the blockage marker |
| Remove all | Click **Clear all** next to the blockage count badge |

The blockage count badge appears in the sidebar whenever at least one blockage is active. Blockages are **preserved** when you reset the route, so you can re-test the same road conditions with different start/end points.

### How Blockage Avoidance Works

When blockages are present, the router uses a two-pass algorithm:

1. **Pass 1** — fetches the direct A→B route to get the base geometry.
2. For each blockage, computes a perpendicular offset waypoint ~500 m away from the nearest segment of the base route.
3. **Pass 2** — fetches a new route through all the detour waypoints in order, forcing OSRM onto an alternative road around each blockage.

---

## Road Type Exclusions

Three checkboxes in the *Road Conditions* section let you exclude entire road categories:

| Checkbox | Excludes |
|---|---|
| Avoid motorways | Highways / motorways |
| Avoid toll roads | Any tolled road or tunnel |
| Avoid ferries | Ferry crossings |

Checking or unchecking any box immediately recalculates the route if both points are set. Exclusions are passed directly to OSRM via the `&exclude=` parameter and apply on top of any blockages.

---

## Address Search

Two search boxes sit below the travel mode selector — one for the start point, one for the end point.

- Type a place name, street address, or landmark.
- Results appear in a dropdown after a short debounce (420 ms).
- Click a result to place the corresponding marker on the map and trigger routing.
- After placing a marker by clicking the map, the address is automatically filled in via reverse geocoding.

Search is powered by **Nominatim** (OpenStreetMap's free geocoder). Results are global but quality may vary for rural or unnamed roads.

---

## GPS Location

Two small **Use my location** buttons (one per search box) let you use your device's GPS position as the start or end point.

- Clicking the button requests your browser's Geolocation API.
- A pulsing blue dot and accuracy circle appear on the map at your position.
- The nearest address is reverse-geocoded and filled into the search box.
- Routing triggers automatically once both points are set.

> **Note:** The browser will ask for location permission the first time. GPS accuracy depends on your device and environment.

---

## Map Styles

A dropdown in the top-right corner of the map lets you switch between three basemap styles:

| Style | Provider | Best for |
|---|---|---|
| **Light** (default) | CARTO | General use, easy to read |
| **Dark** | CARTO | Low-light environments, night mode |
| **Satellite** | Esri World Imagery | Aerial/satellite imagery |

The route line (blue) and markers remain visible on all three styles.

---

## Turn-by-Turn Directions

After a route is calculated, a **Directions** panel appears at the bottom of the sidebar.

Each step shows:
- An **SVG maneuver icon** (colour-coded: blue body, red arrowhead) indicating the turn type.
- A **text instruction** (e.g. "Turn left on", "Continue straight on", "Roundabout — exit on").
- The **road name** for that segment.
- The **distance** for that step (metres or kilometres).

Supported maneuver types:
- Depart / Arrive
- Straight, Slight left/right, Left/Right, Sharp left/right
- Roundabout / Rotary
- U-turn

Directions are fetched in parallel with the route geometry so they appear shortly after the route line, without blocking it.

---

## Route Information Panel

Once a route is drawn, a stats panel appears in the sidebar showing:

| Stat | Description |
|---|---|
| **Distance** | Total route length in kilometres |
| **Est. Time** | Estimated travel time (varies by mode) |
| **Blockages** | Number of blockages avoided, or "—" if none |

The time label changes with the travel mode: *Est. Drive Time*, *Est. Cycle Time*, or *Est. Walk Time*.

---

## Technical Details

### Routing

- **Engine:** [OSRM](https://project-osrm.org/) public demo server (`router.project-osrm.org`)
- **Geometry format:** GeoJSON (`geometries=geojson`)
- **Parallel fetching:** Geometry and turn-by-turn steps are fetched simultaneously. The route line is drawn as soon as geometry arrives; directions populate once steps arrive.
- **Race condition prevention:** An `AbortController` cancels any in-flight request when a new one is triggered (e.g. dragging a marker quickly).

### Geocoding

- **Forward geocoding:** Nominatim (`nominatim.openstreetmap.org/search`)
- **Reverse geocoding:** Nominatim (`nominatim.openstreetmap.org/reverse`)
- Results are debounced at 420 ms to avoid spamming the API.

### Map

- **Library:** [Leaflet 1.9.4](https://leafletjs.com/)
- **Coordinate system:** WGS84 (EPSG:4326)
- **Coordinate order:**
  - Leaflet API: `[latitude, longitude]`
  - OSRM URLs and GeoJSON: `[longitude, latitude]`
- **Direction arrows:** ~7 SVG arrowhead markers placed at even intervals along the route, each rotated to the bearing of travel using the Haversine bearing formula.
- **Fit-to-route:** The map snaps instantly (no animation) to fit the route within 50 px padding after each calculation.

### File Structure

```
shortest-route-map.html   — The entire application (HTML + CSS + JS, ~1 000 lines)
```

No build tools, no bundler, no node_modules, no server.

---

## Dependencies

All loaded from CDN at runtime — no local installation needed:

| Library | Version | Purpose |
|---|---|---|
| [Leaflet](https://leafletjs.com/) | 1.9.4 | Interactive map rendering |
| [CARTO Basemaps](https://carto.com/basemaps/) | — | Light and dark tile layers |
| [Esri World Imagery](https://www.arcgis.com/) | — | Satellite tile layer |
| [OSRM Demo Server](https://router.project-osrm.org/) | — | Road routing API |
| [Nominatim](https://nominatim.openstreetmap.org/) | — | Forward and reverse geocoding |

No JavaScript frameworks, no build step, no package manager.

---

## Limitations

- **OSRM demo server** is a public, rate-limited service. Heavy use or large numbers of simultaneous users may result in slower responses or errors. For production use, host your own OSRM instance.
- **Nominatim** is also a public service with a usage policy (max 1 request/second). The built-in debounce helps, but avoid automated bulk searches.
- **Satellite tiles** (Esri) are subject to Esri's terms of service.
- Blockage avoidance is an approximation — OSRM has no native "avoid this point" feature, so the perpendicular waypoint method may not work perfectly for all road geometries (e.g., dead ends, one-way streets).
- The app requires an internet connection to load tiles, fetch routes, and geocode addresses.
- GPS accuracy depends on the user's device and browser permissions.
