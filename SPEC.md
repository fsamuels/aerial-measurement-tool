# Aerial Measurement Tool — Spec

## Goal

Turn drone photos (and existing satellite imagery) of a 24-acre horse farm into rough
measurements — fence line length and paddock/pasture acreage — and maintain those layers
over time so changes to the property can be tracked and time-lapsed.

**Accuracy target: rough, not survey-grade.** This tool is for planning fence material
orders and tracking acreage, not for legal boundary surveys. A few percent of error on a
fence run or a pasture's acreage is acceptable; sub-meter absolute accuracy is not a goal
and isn't worth the added complexity (ground control points, RTK, etc.) it would require.

**Build-vs-buy:** much of this exists off the shelf — free measurement on satellite
imagery (Google Earth Pro), land-mapping apps (Land id, onX, fieldmargin), and drone
photogrammetry platforms (WebODM, DroneDeploy, Pix4D). See
[docs/product-review.md](docs/product-review.md) for the market survey, pricing, and its
recommendation to adopt WebODM rather than build M1–M4. The milestones below have not yet
been revised against it.

## Problem Statement

- The farm has no accurate map of fence lines or paddock boundaries — estimates today are
  guesswork, which makes ordering electric fence wire/posts inefficient (over- or
  under-buying).
- Drone photos are captured periodically (~120 m altitude) but currently just sit as loose
  photo sets — nothing stitches them into a usable map or lets anyone measure on them.
- Existing satellite imagery (e.g. Esri/USGS) is a reasonable baseline but is out of date
  and doesn't reflect recent fence changes, cleared areas, or new structures.
- There's no record of how the property has changed over time (new paddocks, cleared
  brush, moved fence lines) — the interesting long-term goal is a timelapse of the
  property built from successive drone flights layered over the satellite baseline.

## Non-Goals (v1)

- Survey-grade / legal-boundary-accurate measurement.
- Real-time flight planning or drone control.
- Automatic fence detection via computer vision (fences are drawn manually on the map for
  now — see [Open Questions](#open-questions)).
- Multi-property / multi-user support — this is a single-farm, single-user tool.

## Architecture

Two decoupled pieces, because they have very different hardware needs:

```
┌─────────────────────┐        orthomosaic         ┌───────────────────────┐
│   stitcher (CLI)     │  ────────GeoTIFF─────────▶ │   viewer (web app)    │
│                      │        + metadata          │                       │
│ Runs on your main     │                            │ Runs anywhere light   │
│ computer — needs real │                            │ (laptop, home server, │
│ CPU/RAM for photo-    │                            │ eventually remote) —  │
│ grammetry.            │                            │ just serves map tiles │
│                       │                            │ + a small database.   │
└─────────────────────┘                              └───────────────────────┘
```

- **`stitcher`** — a CLI tool you run manually after a drone flight. Takes a folder of
  geotagged flight photos, runs them through a photogrammetry pipeline, and produces a
  georeferenced orthomosaic (a single top-down, map-aligned image) plus flight metadata
  (date, photo count, altitude, ground coverage area).
- **`viewer`** — a small self-hosted web app. Shows a map with a satellite basemap, lets
  you toggle any stitched orthomosaic on top of it, draw and label fence lines and paddock
  boundaries, and computes their real-world length/area. Built to run comfortably on a
  laptop or low-power home server now, with no architectural changes needed if it later
  moves to a more powerful machine or gets hosted remotely (see
  [Deployment](#deployment)).

They talk to each other only through files: the stitcher writes an orthomosaic + a
metadata JSON sidecar into a shared `data/flights/<flight-id>/` directory; the viewer
reads that directory. No API between them, no shared process.

## Photogrammetry Pipeline (`stitcher`)

- **Tool:** [OpenDroneMap (ODM)](https://github.com/OpenDroneMap/ODM), run via its Docker
  image (`opendronemap/odm`). Not reimplementing structure-from-motion — ODM is
  well-established, free, and handles GPS-tagged drone photos out of the box.
- **Input:** a folder of JPEGs from one flight, each with EXIF GPS + altitude (standard
  for consumer drones at 120 m AGL). No ground control points in v1 — relying on the
  drone's onboard GPS, consistent with the rough-accuracy goal above.
- **`stitcher` CLI responsibilities** (thin wrapper around ODM, not a reimplementation):
  1. Validate the input folder — enough photos, all geotagged, altitude roughly
     consistent (catches a bad/partial flight import early rather than failing deep
     inside ODM after a long run).
  2. Run ODM via Docker with a fixed, documented parameter set tuned for orthophoto
     output (not full 3D reconstruction — that's slower and unnecessary for this use
     case).
  3. On success, copy the resulting orthophoto GeoTIFF and ODM's report into
     `data/flights/<flight-id>/`, alongside a `flight.json` sidecar (date, photo count,
     altitude, bounding box, processing duration).
  4. Print a summary (coverage area, resolution) so you can sanity-check the flight
     before opening the viewer.
- **Runtime expectation:** minutes to a few hours depending on photo count and machine —
  this is why it's a separate CLI step run on your main computer, not something the
  always-on viewer does.

## Web Viewer (`viewer`)

- **Map:** [MapLibre GL](https://maplibre.org/) (open-source, no vendor lock-in) or
  Leaflet — final choice deferred to implementation (see
  [Open Questions](#open-questions)); either renders a satellite basemap with the
  orthomosaic as a togglable/opacity-adjustable overlay layer.
- **Basemap:** free tile sources — USGS NAIP (public domain, good baseline for US
  agricultural land) as primary, Esri World Imagery as a fallback/alternate. No paid tile
  provider in v1, per the earlier decision to keep this free to run.
- **Flight layers:** every stitched flight in `data/flights/` appears as a selectable
  overlay, with a date label and opacity slider, so a drone orthomosaic can be compared
  directly against the satellite baseline or against an earlier flight.
- **Measurement tools:**
  - Draw a polyline → labeled as a fence run → length computed geodesically (not naive
    Euclidean lat/lon, which distorts meaningfully even at this scale) and shown in feet.
  - Draw a polygon → labeled as a paddock/pasture → area computed geodesically, shown in
    acres.
  - Every measurement is saved (label, geometry, computed value, which flight it was
    drawn against, timestamp) — not just a scratch calculation that disappears on reload.
  - A measurements list/table view, editable and deletable.
- **Timelapse:** given 2+ flights, generate a video/gif stepping through the orthomosaics
  in date order (same approach as the existing `timelapse-creator` / `bluewood-timelapse`
  projects — `ffmpeg` over a sequence of images), cropped/aligned to a consistent extent
  so the property doesn't visually "jump" between frames.

## Data Model

SQLite for v1 — a single-user local tool doesn't need a database server, and it's a clean
upgrade path to Postgres/PostGIS later if this moves to a real server (per the "hosted
remotely" note in Deployment).

| Table | Purpose |
|---|---|
| `flights` | One row per stitched drone flight: id, date, altitude, photo count, orthomosaic path, bounding box, processing duration |
| `measurements` | Fence/paddock geometries: id, type (`fence`\|`paddock`), label, geometry (GeoJSON), computed value (length or area), unit, flight_id (nullable — a measurement can be drawn against the satellite baseline with no flight), created/updated timestamps |

Orthomosaic GeoTIFFs and raw flight photos are **not** committed to git (large binary
data) — `data/` is gitignored, matching the `archive/` convention in `timelapse-creator`.

## Deployment

Starts as a fully local setup: `stitcher` run by hand on your main computer, `viewer` run
as a local web server (`docker compose up`, viewed at `localhost`) on whatever machine is
convenient — laptop or a home server if you want it always-on. No cloud dependency, no
ongoing cost.

Designed so growth doesn't require a rewrite:
- If flight processing outgrows your main computer, `stitcher` can run unchanged on a
  more powerful machine (or a cloud VM) — it only needs the input photo folder and Docker.
- If you want the viewer reachable outside your home network, it can be reverse-proxied or
  hosted on a small remote server — it's a stateless-except-SQLite web app, no code
  changes needed, just where it runs and how it's exposed.

## Tech Stack

Matches conventions already used across other farm projects (`electric-fence-monitor`,
`timelapse-creator`) rather than introducing a new stack for its own sake:

- **`stitcher`:** Python CLI, calls ODM via Docker.
- **`viewer` backend:** Python, FastAPI.
- **`viewer` frontend:** React + TypeScript, MapLibre GL or Leaflet for the map.
- **Storage:** SQLite (measurements/flight metadata) + filesystem (orthomosaics, gitignored).
- **Run:** Docker Compose (`stitcher`'s ODM container + `viewer`'s frontend/backend/db
  services), consistent with `electric-fence-monitor/dashboard`'s compose setup.

## Repository Layout (planned)

```
SPEC.md                   This file
README.md                 Project overview, points here + to docs/
CLAUDE.md                 Working rules (branching, commits) — links to sdlc-standards
stitcher/                 CLI: photo validation, ODM wrapper, flight.json output
viewer/
  backend/                FastAPI app: flights + measurements API, SQLite
  frontend/                React/TS app: map, layers, drawing tools, measurements list
data/                     Gitignored: flight photos in, orthomosaics + SQLite db out
docs/
  architecture.md         Current-state architecture (created once something is built)
  current-status.md       What's done vs. open
  roadmap.md              Prioritized upcoming work
  open-questions.md       Decisions still open (see below — starting point)
```

## Milestones (module-by-module)

Per the [sdlc-standards](https://github.com/fsamuels/sdlc-standards) build-out
convention — each implemented, tested, and merged independently:

1. **M1 — `stitcher` MVP:** wrap ODM, validate input, produce one orthomosaic +
   `flight.json` from a real test flight's photos. Proves the hardest/riskiest part
   (does ODM handle this drone's photos and altitude well) before any viewer UI exists.
2. **M2 — `viewer` skeleton:** map with satellite basemap only, no flight data yet.
   Confirms the map/tile stack works before layering in flight data.
3. **M3 — Flight layers:** load `data/flights/` into the viewer, toggle/opacity-control
   orthomosaic overlays against the basemap.
4. **M4 — Measurement tools:** draw/save/edit/delete fence (length) and paddock (area)
   measurements, backed by SQLite.
5. **M5 — Timelapse:** generate a video/gif across 2+ flights once enough flight history
   exists.

## Open Questions

- [ ] **Photo overlap check:** does the current drone flight pattern have enough
  front/side overlap (~75–80%/60–70% is typical for good photogrammetry results) for ODM
  to stitch reliably? Needs checking against a real photo set before M1 is considered
  proven, not just assumed.
- [ ] **Flight-to-flight alignment for timelapse:** successive flights won't have
  pixel-identical extents/altitude — does the timelapse crop to the smallest common
  bounding box, or is some alignment step needed? Decide once 2+ real flights exist to
  test against (M5).
- [ ] **MapLibre vs Leaflet:** both work for this; pick based on whichever has better
  GeoTIFF/COG overlay support with least friction when M2 starts.
- [ ] **Drone/EXIF specifics:** which drone model, and does it write standard EXIF GPS +
  relative altitude tags ODM expects? Confirm before M1.
- [ ] **Existing fence hardware data:** should `electric-fence-monitor`'s node locations
  eventually appear on this map too (shared farm-map context), or stay fully separate
  projects? Deferred — not needed for v1, worth revisiting once both tools exist.
