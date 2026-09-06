# Product Review — Existing Tools That Overlap This Spec

**Date:** 2026-09-01 · **Status:** decision input for [SPEC.md](../SPEC.md), not a
commitment to build or buy.

## Why this doc exists

The question raised: *"there may be a lot of existing software to do this or similar — I
found I can just use Google Maps for the distance roughly based on existing satellite
imagery."* That's correct, and it deserves a real answer before M1 gets built. This is a
survey of everything on the market that covers part or all of the spec, with features and
pricing as of September 2026, ending in a build-vs-buy recommendation.

## What we're actually comparing against

SPEC.md asks for five capabilities. Almost no product does all five; most do one or two.
Scoring against these is what the tables below do:

| # | Capability | Notes |
|---|---|---|
| **C1** | Measure fence length (geodesic polyline, feet) on satellite imagery | Widely available, including free |
| **C2** | Measure paddock acreage (geodesic polygon, acres) | Widely available, including free |
| **C3** | **Save/label/edit** measurements persistently, not scratch calculations | Narrows the field considerably |
| **C4** | Stitch own drone photos → georeferenced orthomosaic | Photogrammetry products only |
| **C5** | Toggle successive own-flight orthomosaics over a basemap → **timelapse of the property over time** | Effectively nobody does this cheaply |

Also relevant: the accuracy target is explicitly *rough, not survey-grade*. That
disqualifies the expensive half of the drone market on value grounds before features are
even compared — paying survey-grade prices for a stated non-survey use case is the main
way this project could waste money.

---

## Tier 1 — Free measurement on existing satellite imagery

This tier alone covers C1 and C2, which is the bulk of the day-to-day value (ordering
fence wire, knowing a paddock's acreage).

| Product | What it does | Covers | Pricing |
|---|---|---|---|
| **Google Maps (desktop web)** | Right-click → *Measure distance*, click points; close the shape and it reports area. Mobile app does distance but not closed-shape area. | C1, C2 | Free |
| **Google Earth Pro (desktop)** | Ruler tool with paths and polygons, in feet/acres. **Saves** paths and polygons as named places in a sidebar, exports KML/KMZ. Historical imagery slider (clock icon) steps back through years — decades in some areas. Image overlays let you drape your own georeferenced image over the basemap. | C1, C2, **C3**, partial **C5** | Free |
| **Free web acreage calculators** (mapwithradius, simplemaplab, and similar) | Draw a polygon on a Google/Esri basemap, get acres/ft²/perimeter. Geodesic math. Some keep multiple colored shapes with a running total and export KML/PNG. | C1, C2, weak C3 | Free, ad-supported |
| **MapMagician suite** ([mapmagician.in](https://www.mapmagician.in/)) — the site referenced | Three Android apps: *Development Plan GIS* (Indian zoning overlays + measuring), **Overlayr** (georeference a scanned map/site plan onto satellite imagery via control points, measure on it, export KMZ), *Location Plan Maker Pro* (site plans with markers, PDF export). | C1, C2, partial C3 | Free tiers; Overlayr Premium unlocks unlimited projects |
| **QGIS** | Full desktop GIS. Measure tool, geodesic area, loads GeoTIFF/COG orthomosaics directly, unlimited layer management, NAIP/Esri basemaps via XYZ tiles. Steep learning curve, not a "farm app." | C1–C3, and **displays** C4 output; C5 only manually | Free, open source |

**Note on MapMagician specifically:** it's an India-focused zoning/plan-overlay suite. The
one genuinely transferable idea is **Overlayr** — control-point georeferencing of an
arbitrary image onto satellite imagery. That's the poor-man's version of the whole
`stitcher` → viewer overlay pipeline: if a stitched orthomosaic ever needs nudging into
place by hand, this is the pattern. The zoning-plan products have no US/farm relevance.
The site also disclaims government affiliation and says data is "for reference only."

**Google Earth Pro is the strongest free option and closest to a free version of the
viewer** — saved named polygons, feet/acres, KML export, historical imagery for change
tracking, and image overlays for draping a drone orthomosaic. Its weaknesses versus the
spec: no measurement database or table view, no opacity slider workflow across many
flights, and no video export.

---

## Tier 2 — Land / farm mapping apps (the closest commercial match to `viewer`)

These are what the spec's viewer would be if someone else built it: persistent labeled
maps, measuring, parcel data, mobile access.

| Product | Features | Covers | Pricing (2026) |
|---|---|---|---|
| **Land id** (formerly MapRight) | Nationwide parcel/ownership data, 7 basemaps + 40 overlay layers, distance & area measurement, custom lines/polygons, 40+ markers, 3D terrain, photo waypoints, hi-res aerial imagery, offline mobile, map sharing links, hi-res printing, deed plotting, soil reports. | C1, C2, **C3**; no C4/C5 | Basic **$7/mo** annual ($15 monthly); **Premium $12/mo** annual ($20 monthly) — adds 3 custom maps, hi-res imagery, markup tools; **Pro $33.33/mo** (3 maps) or **$66.67/mo** (unlimited) annual |
| **onX Hunt** | Line Distance and Area Measure tools, 147M private parcels with owner name/acreage, waypoints, tracks, offline maps, historical/multiple imagery sources. Consumer-grade and genuinely good at exactly the measure-and-save loop. | C1, C2, C3 | Premium **$34.99/yr** (1 state), $49.99/yr (2 states), **Elite $99.99/yr** (50 states + Canada) or $14.99/mo |
| **onX Backcountry** | Same mapping engine, recreation-focused; private parcel/acreage data only at Elite. | C1, C2, C3 | Premium $29.99/yr; Elite $99.99/yr |
| **fieldmargin** | "Living map" of the farm — draw and measure fields by drawing or by walking GPS, unlimited team members, field notes/jobs, livestock, offline sync, **and it accepts drone and satellite imagery layers**. Genuinely farm-shaped rather than hunt-shaped. | C1–C3, partial C4-display | **Free** tier; Essentials **£11.99/mo**; Plus £29.99/mo; Pro £53.99/mo |
| **Acres.com** | Parcel-level land GIS, plat maps, sold-land comps, acreage search. Land-investment oriented. | C2, parcel data | From ~**$7.49/mo**; Acres+/Pro/Enterprise tiers |
| **Regrid** | 160M parcels, property app plus bulk parcel data licensing by county/state. Data source more than a viewer. | Parcel data | Volume-based; county/state downloads |
| **Felt** | Modern collaborative web GIS — upload GeoTIFF, draw, measure, share. Good orthomosaic viewer without QGIS's learning curve. | C1–C3, displays C4 | Free tier; paid from ~**$200/mo** flat |

**Land id Premium at $12/month is the single closest commercial product to the spec's
viewer** — labeled, saved, measurable maps with hi-res imagery. **fieldmargin's free tier
is the closest farm-native one**, and it explicitly supports adding drone imagery layers.

---

## Tier 3 — Drone photogrammetry (the `stitcher` half)

This is where the money is, and where the accuracy target matters most. Prices below are
for products aimed at survey/construction/precision-ag customers — a 24-acre horse farm
wanting rough numbers is far below their intended buyer.

| Product | Model | Covers | Pricing (2026) |
|---|---|---|---|
| **WebODM Lightning** (hosted OpenDroneMap) | Upload photos → orthomosaic, DEM, point cloud, 3D. Volume/elevation/area measurement built in. GeoTIFF/LAZ export. Note: WebODM separated from OpenDroneMap in April 2026 and now runs **ODX**, a fork of the ODM engine. | **C4**, C1–C3 in-app | **Starter $24/mo**, **Pro $35/mo** (unlimited maps, 1,500 img/map, 100 GB), Business $99/mo (3,000 img, 1 TB). Pay-as-you-go credits; 150 free on signup. Pausable up to 120 days. |
| **Maps Made Easy** | Browser-based cloud stitching, pay-per-use points or subscription. | C4 | **Free** for unlimited jobs up to 325 images @ 12 MP; points beyond that |
| **DroneDeploy** | The market leader. Flight planning, cloud processing, annotations, measurement, progress tracking, AI analytics. | C4, C1–C3, partial C5 (dated map history) | **Ag Lite $1,908/yr** (1,000 img/map); **Flight & Analysis $4,188/yr** (3,000 img/map); Advanced custom |
| **Pix4Dmapper** | Desktop photogrammetry, survey-grade, GCP support. | C4 | ~**$3,990/yr** subscription; perpetual license **~$14,990** as of Jan 2026 (up from ~$5,990) |
| **Pix4Dfields** | Ag-focused; credit system priced on satellite imagery area (0.75 credits/ha for 30–70 cm imagery). | C4, ag indices | Credit-based; see Pix4D pricing |
| **DJI Terra** | Desktop 2D/3D from DJI drones, perpetual or annual licence, LiDAR support. | C4 | Annual from **~$999/yr**; perpetual licences sold per device by resellers |
| **Propeller Aero** | Earthworks/mining, per-project pricing, AeroPoint GCP hardware. | C4, survey-grade | Not published — sales call |
| **Solvi** | Ag analytics on drone imagery, per-hectare. | C4 + plant analytics | €3.50/ha basic counts; €2/ha weed detection; €30 minimum per job |
| **Self-hosted OpenDroneMap / WebODM** | Docker, unlimited local processing, GeoTIFF/LAZ out. **This is exactly what SPEC.md's `stitcher` already proposes wrapping.** | **C4** | **Free**; needs ≥16 GB RAM, ~100 GB disk, and your own time |

**The spec is already correct here.** It picked ODM via Docker rather than reimplementing
photogrammetry. The finding is not that ODM is wrong — it's that **WebODM (self-hosted, or
Lightning at $24–35/mo) is a finished product that already includes a web UI, layer
management, and measurement tools**, which is most of what M2–M4 propose building.

---

## What nothing on this list does well

**C5 — a timelapse of your own successive orthomosaics.** DroneDeploy keeps dated maps you
can flip between; Google Earth Pro has a historical-imagery slider but only for *its*
imagery, not yours. Nothing surveyed produces a video/gif stepping through your own
flights cropped to a consistent extent. That's a genuine gap — and it's also the one part
of this spec that is already 80% solved in another repo (`timelapse-creator` /
`bluewood-timelapse`, ffmpeg over an image sequence).

**Fence-material-oriented output** (a labeled fence run list totalling linear feet for
ordering wire and posts) is likewise absent — every product gives you a measurement, none
gives you a bill of materials. Small feature, but it's the actual problem statement.

---

## Cost comparison at this farm's scale

| Approach | Year 1 | Ongoing/yr |
|---|---|---|
| Google Maps / Earth Pro only | $0 | $0 |
| fieldmargin free tier | $0 | $0 |
| onX Hunt Elite | $100 | $100 |
| Land id Premium | $144 | $144 |
| Self-hosted WebODM + Google Earth Pro | $0 (hardware you own) | $0 |
| WebODM Lightning Pro | $420 | $420 |
| Build SPEC.md as written | $0 cash, ~5 milestones of build time | maintenance |
| DroneDeploy Ag Lite | $1,908 | $1,908 |
| Pix4Dmapper perpetual | $14,990 | — |

---

## Recommendation

**Don't build M1–M4. Build only what's actually missing.**

The honest reading of this survey: the spec proposes rebuilding, from scratch, things that
exist for free or for ~$12/month, in order to eventually get one feature (C5) that nobody
sells. Specifically:

1. **Measurement (C1–C3) — buy or use free, don't build.** Google Earth Pro already saves
   labeled paths and polygons in feet and acres and exports KML, at $0. If saved,
   shareable, mobile-accessible maps matter, **Land id Premium ($144/yr)** or **onX Hunt
   Elite ($100/yr)** are finished products that beat what M2–M4 would produce. This
   retires the entire viewer build.

2. **Stitching (C4) — use WebODM as a product, not as a library.** SPEC.md's `stitcher`
   is a thin wrapper around ODM; **self-hosted WebODM is that wrapper, already written**,
   with a web UI and measurement tools attached, for free. Lightning at $24–35/mo removes
   even the hardware requirement. M1's real value was never the wrapper code — it was
   answering the open question *"does ODM handle this drone's photos at 120 m?"*, and
   WebODM answers that in an afternoon rather than a milestone.

3. **Timelapse (C5) — this is the only part worth building**, and it should be a small
   script, not a platform: export orthomosaic GeoTIFFs from WebODM, crop to a common
   bounding box, run the existing `timelapse-creator` ffmpeg pipeline over them. That's
   days of work reusing an existing repo, not five milestones.

4. **Sequence it as a cheap test.** Measure the fence lines in Google Earth Pro this week
   at $0. If the satellite imagery's staleness genuinely produces wrong wire orders, that
   justifies drone flights — then run WebODM. If it doesn't, the whole project was
   correctly avoided and the fence gets ordered anyway.

**The one argument for building anyway** is that this is a hobby farm-software portfolio
alongside `electric-fence-monitor` and `timelapse-creator`, where the build *is* the point
and vendor lock-in/subscription avoidance is a real preference. That's a legitimate reason
— but it should be stated in SPEC.md as the motivation, rather than the spec implying no
alternatives exist. If that's the call, the scope should still shrink: adopt WebODM for
M1, and spend the effort on the fence-materials output and timelapse, which are the two
things the market genuinely doesn't sell.

---

## Sources

- [MapMagician](https://www.mapmagician.in/) · [Google Earth measurement docs](https://support.google.com/earth/answer/9010337) · [Google historical imagery](https://newsinitiative.withgoogle.com/resources/trainings/google-historical-imagery-google-earth-maps-and-timelapse/)
- [Land id pricing](https://id.land/pricing) · [onX Hunt pricing](https://www.onxmaps.com/hunt/app/pricing) · [onX Backcountry pricing](https://www.onxmaps.com/backcountry/app/pricing) · [fieldmargin](https://www.capterra.com/p/163565/fieldmargin/) · [Acres pricing](https://www.acres.com/pricing) · [Regrid plans](https://app.regrid.com/plans) · [Felt pricing](https://felt.com/pricing)
- [WebODM Lightning pricing](https://webodm.net/pricing) · [WebODM vs OpenDroneMap, 2026 fork](https://www.skyebrowse.com/news/posts/webodm-vs-opendronemap) · [DroneDeploy pricing](https://www.dronedeploy.com/pricing) · [PIX4Dmapper pricing](https://www.pix4d.com/pricing/pix4dmapper/) · [PIX4Dfields pricing](https://www.pix4d.com/pricing/pix4dfields/) · [DJI Terra 2026 versions & pricing](https://www.terrestrialimaging.com/blogs/news/dji-terra-in-2026-versions-pricing-and-whats-new) · [Solvi pricing](https://solvi.ag/pricing) · [Maps Made Easy free tier](https://www.skyebrowse.com/news/posts/free-drone-mapping-software) · [Drone mapping software cost comparison](https://dronelaunchacademy.com/resources/best-drone-mapping-software/)

Prices are list prices as published in September 2026 and change without notice; anything
gated behind a sales call (Propeller, DroneDeploy Advanced/Team) is noted as such.
