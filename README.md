# Waypoints: A Personal BioGeography
 
![GitHub last commit](https://img.shields.io/github/last-commit/johbry17/waypoints)

*Mapping memories: An interactive travelogue handcrafted with Leaflet.js, Python, and open-source geospatial resources.*

🌍 [Interactive Atlas](https://johbry17.github.io/Waypoints/)

> ⚠️ This project is under active development.

## Table of Contents

- [Overview](#overview)
- [Gallery](#gallery)
- [Features & Interactions](#features--interactions)
- [How It Works](#how-it-works)
- [Tools & Technologies](#tools--technologies)
- [Usage](#usage)
- [Data Documentation](#data-documentation)
- [References](#references)
- [Licenses](#licenses)
- [Acknowledgements](#acknowledgements)
- [Author](#author)

## Overview

**Waypoints: A Personal BioGeography** is a map-based storytelling project—an interactive memoir built from travels, memories, and photos. The map spans five continents (for now!): India's temples and backwaters, Central America's volcanoes and reef systems, Namibia's desert canyons, Peru's highland rail lines, Scotland's coastline, Bali's volcanic peaks, and dozens of places in between. Each waypoint opens a photo carousel; overlaid routes trace how those places connect by plane, train, car, boat, and foot.

What began as a simple way to pin photos on a map evolved into a data project: a Python pipeline reads travel records from Google Sheets, geocodes locations, generates route geometries, and produces the static files that power the map.

The current build includes:
- **47 major waypoints** with photo carousels and personal descriptions
- **327 routes** — 197 road trips, 47 flights, 34 boat journeys, 26 hikes, 23 train trips
- **35 activities** (hike, snorkel, safari, kayak, and more)
- **6 landmark locations** (future expansion layer)

[View Data Dictionary](data_dictionary.md)

## Gallery

![World map with color-coded markers identifying major locations across the author's travels.](./resources/images/main_markers.png)
*The atlas overview uses color-coded markers to distinguish life imprints. Interactive popups connect individual locations to their associated photographs and narratives.*

![India location popup with a photographic reel and commentary, displayed over the map's route layer.](./resources/images/photo_reel_india.png)
*A photo-and-commentary popup viewed alongside the route layer, linking a specific travel experience to its geographic context.*

![Map of Central America showing an Ometepe hiking popup, activity markers, waypoint markers, travel routes, legends, and map controls.](./resources/images/activities_central_america.png)
*An activity popup appears over a map populated with waypoints, activity markers, and travel routes. The view brings together location-specific details and the atlas's interactive controls, including route selection, map legends, zoom, About, and reset functions.*

![Namibia location popup displaying a reel of personal photographs and accompanying commentary.](./resources/images/photo_reel_namibia.png)
*A location-level view combining a photo reel with written commentary, integrating personal media into the geographic browsing experience.*

![Map of Vermont showing activity markers and mapped travel routes at a local scale.](./resources/images/activities_vt.png)
*Activity markers and route lines illustrate the atlas's use at a finer geographic scale, from individual outings to broader travel patterns.*

![Map of the US Virgin Islands showing activity markers, a waypoint, and connected travel routes.](./resources/images/activities_stj.png)
*Activity markers and travel routes appear together, showing how the interface handles point features and connected paths at the regional scale.*

<!-- ![Entity Relationship Diagram](./resources/images/ERD.png) -->

## Features & Interactions

**Map layers (all toggleable)**
- **Waypoints** — circle markers scaled by a 1–10 importance score; home residences and academic locations styled distinctly
- **Activities** — point markers with typed icons: hiking, snorkeling, kayaking, safari, paragliding, and more
- **Routes** — polylines for five transport modes, each with its own color and dash pattern, individually toggleable (plane, train, auto, boat, hike)

**Basemaps** — six options: Satellite (ESRI), Street Map (OpenStreetMap), National Geographic, Physical, Oceans, Grayscale

**Popups & photo carousels** — each marker opens a popup with photos, descriptions, and an optional zoom-to-area button; carousels support autoplay, fullscreen, and pinch/scroll zoom

**Locations layer** — landmark and place-type markers that appear automatically above zoom level 6

**Welcome screen & guided tour** — a splash modal on load followed by an optional Shepherd.js tour walking through the map's layers and controls

**Legends** — a main legend for waypoint types; a route-type legend (with line-style previews) appears when the Routes overlay is active

## How It Works

Travel data lives in Google Sheets; a Python script converts it to static files the map reads.

```
Google Sheets (source of truth)
     │
     ▼
scripts/extract_data.py
     │  assigns UUIDs · geocodes coordinates · generates route GeoJSON
     ▼
docs/resources/data/     ← overview.json, activity.json, location.json, routes.csv
docs/resources/geojson/  ← one .geojson per route (331 files)
     │
     ▼
Leaflet.js map (index.html)
     └── fetches JSON/CSV at load · loads GeoJSON per route · renders layers
```

**Python pipeline**

`scripts/extract_data.py` processes four sheets in sequence:

- **UUID assignment** — new rows get a stable UUID so records survive spreadsheet reordering
- **Geocoding** — missing coordinates are resolved via Nominatim; results are cached locally to avoid redundant API calls; ambiguous place names can be overridden in `scripts/manual_locations.py`
- **Route geometry generation** — by transport mode:
  - *Plane*: geodesic great-circle arcs via GeographicLib, with correct handling of the International Date Line
  - *Auto*: driving routes from the OpenRouteService API
  - *Train*: real rail geometry sourced from Overpass Turbo (or a driving-route fallback if no GeoJSON is provided)
  - *Boat / Hike*: manually sourced from Overpass Turbo or hand-drawn with geojson.io
- **Write-back** — enriched data (UUIDs, geocoded coordinates) is uploaded back to Google Sheets by default

**Frontend**

All vendor libraries are bundled locally — no CDN calls at runtime. The Leaflet map:

- Triples each marker at ±360° longitude to handle International Date Line crossings without gaps
- Uses custom Leaflet panes to control z-order precisely across routes, locations, waypoints, activities, and popups
- Parses `routes.csv` with PapaParse and loads each route's GeoJSON on demand
- Draws ESRI basemaps (Satellite, Nat Geo, Physical, Oceans, Grayscale) via Esri Leaflet

## Tools & Technologies

| Layer | Tools |
|---|---|
| **Frontend** | Leaflet.js, Esri Leaflet, Shepherd.js, PapaParse, Panzoom, HTML/CSS/JS |
| **Icons** | Font Awesome, Material Design Icons |
| **Data pipeline** | Python, pandas, gspread, geopy (Nominatim), GeographicLib, openrouteservice |
| **Geospatial sources** | OpenRouteService API, Overpass API, Nominatim, geojson.io |
| **Data formats** | GeoJSON, JSON, CSV |
| **Hosting** | GitHub Pages |

## Usage

**Exploring the map**

The project is live at [johbry17.github.io/Waypoints](https://johbry17.github.io/Waypoints/). No local setup required.

1. Click through the welcome splash to open the map; take or skip the guided tour
2. Click any marker to open its popup and photo carousel
3. Use the layer control (top-right) to toggle Waypoints, Activities, and Routes overlays
4. Within Routes, toggle individual transport modes (plane, train, auto, boat, hike)
5. Switch basemaps from the same layer control

**Updating the data**

Travel data is managed in Google Sheets. To add records and regenerate map files:

1. Edit the relevant tab (Overview, Activity, Location, or Routes) in the spreadsheet
2. For any new `auto` or `train` route whose start/end location geocodes incorrectly, add manual coordinates to `scripts/manual_locations.py`
3. From the project root:
   ```bash
   python scripts/extract_data.py
   ```
4. Review generated files in `docs/resources/`, then commit

See [`scripts/README.md`](scripts/README.md) for CLI options (`--no-upload`, `--sheet-key`, `--verbose`), configuration details, and the full field reference.

> **Configuration**: `scripts/api_keys.json` must contain a Google service-account key (and optionally an OpenRouteService key). This file is excluded from version control.

## Data Documentation

- [`data_dictionary.md`](data_dictionary.md) — table schemas and field definitions for all four data tables (Overview, Activity, Location, Routes)
- [`scripts/README.md`](scripts/README.md) — pipeline walkthrough, CLI reference, geocode cache details, and per-transport-mode behavior

## References

- [Leaflet.js](https://leafletjs.com/) – For rendering interactive maps with custom markers and polylines
- [Overpass Turbo](https://overpass-turbo.eu/) – For querying OpenStreetMap data (trails, routes, POIs)
- [Nominatim API](https://nominatim.org/release-docs/latest/) – For geocoding and reverse geocoding
- [OpenRouteService API](https://openrouteservice.org/) – For automatically generating road trip routes
- [GeographicLib](https://geographiclib.sourceforge.io/) – For geodesic calculations (e.g., great circle routes)
- [geojson.io](https://geojson.io/) – For hand-drawing and editing GeoJSON routes
- [GeoJSON Specification](https://geojson.org/) – Core format for encoding spatial data
- [GitHub Pages](https://pages.github.com/) – Hosting platform
- [Google Sheets API](https://developers.google.com/sheets/api) and [Python](https://www.python.org/) – For managing and updating project data
- [Shepherd.js](https://shepherdjs.dev/) – For building the interactive guided tour
- [Font Awesome](https://fontawesome.com/) and [Material Design Icons](https://materialdesignicons.com/) – Iconography and styling assets

## Licenses

- **Code**: MIT License – see [LICENSE](LICENSE)
- **Images**: Personal content not for reuse – see [LICENSE_IMAGES](LICENSE_IMAGES.md)

> **If you're in a photo and want it removed, contact me: bryan.johns@informedwanderer.com**

## Acknowledgements

Thanks to everyone who's shared this beautiful world with me. And to the open-source community for the tools that made this project possible.

## Author

Bryan Johns  
Last updated: <!-- START_DATE -->October 2026<!-- END_DATE -->  
[bryan.johns@informedwanderer.com](mailto:bryan.johns@informedwanderer.com) | [LinkedIn](https://www.linkedin.com/in/b-johns/) | [GitHub](https://github.com/johbry17) | [Portfolio](https://informedwanderer.com)  
— Fluent in Data. Fluent in Human.

