# Changelog — Competitor Entitlement Reporter

Standalone single-file ArcGIS app. Live: https://wildhare1966.github.io/entitlement-reporter/
Sessions are labeled S1, S2, … (the app's own dedicated sessions).

## S2 — 2026-06-30
Google Earth integration (KML export + one-tab follow), an Esri 3D toggle, and detail-panel cleanup.

### Added
- **🌎 Earth KML export** (rail button `#kmlbtn`): one click exports **all** tracked projects (the 4
  statuses) to a styled `LeadsDeals_Arbor.kml` for Google Earth, mirroring the app —
  one KML `<Style>` per status from `STATUS_SYMBOLS`, a `<description>` field table per placemark from
  `DETAIL_FIELDS` (with the `Hearing_Date` UTC fix + `LINK_FIELDS` as links), WGS84 coords, multipart
  polygons + holes handled. Verified live: 452 placemarks / 4 styles / 646 polygons. Helpers:
  `buildKml`, `geomToKml`, `kmlColor`, `descTable`, `ringIsCW`.
- **3D toggle** (rail button `#view3dbtn`, ⛰ 3D ⇄ 🗺 2D): swaps the map between Esri `MapView` (2D) and
  `SceneView` (3D globe + world terrain) — **free, no API key**. `buildView(use3D)` reuses the *same*
  `map`/layers, preserves the camera (`viewpoint`), and re-wires the click handler via `wireView()`.
  Tracked Projects render as **status-colored extruded blocks** in 3D (`build3DRenderer`, 120 m,
  `polygon-3d`/`extrude`), flat fills in 2D.
- **Earth "follow" behavior:** the 🌎 Google Earth detail button now reuses **one** Earth tab
  (named `ge_arbor`) and re-points it to each record you select in the list (`openEarth`/`followEarth`),
  instead of spawning a new window per click.

### Fixed
- **`view.destroy()` was tearing down the shared `map`** ("The provided map is already destroyed") when
  toggling views → detach first (`view.map = null`) before destroy.

### Changed
- Detail-panel Google buttons: **removed 🛰 Google Maps**; **renamed 🌎 Earth → 🌎 Google Earth**; Earth
  camera pulled back `300d → 4525d` (still top-down `0t`) to frame the whole site (per operator screenshot).
- Click handler extracted to hoisted `onMapClick()` so it re-attaches to whichever view is active.

### Researched (not built — see kickoff)
- Syncing `LeadsDeals_Arbor` *into* Google Earth: KML is the only ingestion path; **Earth Web** can't
  load a local file/URL or auto-refresh; **Earth Pro** supports NetworkLink (live). Autonomous Drive
  upload is blocked by the connector's inline-content size limit (~170k tokens for 452 polygons).
- "Rebuild the reporter UI in Google Earth": not possible in the consumer app (no SDK); a Google
  *photorealistic 3D* version is possible via Maps JS `Map3DElement`/Cesium but needs a key + billing +
  a rewrite. Esri `SceneView` (now shipped) was the free 90/10 win.

HEAD at end of S2: latest commit on `main` (run `git log -1`).

## S1 — 2026-06-29
Spun out of `indiana-mip-tracker` into its own repo + GitHub Pages, then patched and extended.

### Fixed
- **Dead field references** (rendered blank): `Description→Terms`, `Owner_Name→OwnerName`,
  `Broker_Contact→Broker`, verified against the live `LeadsDeals_Arbor` schema.
- **`Hearing_Date` date display**: field is type `date-only`; `dateFields` only caught `date` →
  now matches `/date/`. `fmtDate()` switched to **UTC getters** (date-only is UTC-midnight → local
  getters rolled back a day).
- **BLC LTM theme**: `Sale_Price` doesn't exist → graduated color now on `Close_Price`.

### Added
- **`rf()` fuzzy fallback** (norm-equal then substring) as a schema-drift safety net; exact match
  still wins, so no behavior change against today's schema.
- **Google deep-link buttons** in the detail panel (no API key, ToS-clean):
  - 🛰 Google Maps (satellite, z18), 🌎 Earth (top-down ~300 m), 👁 Street View.
  - Street View uses a **click-to-place pick mode** (hint banner + crosshair + Esc/Cancel) because the
    parcel centroid is inside the lot where no panorama exists.

### Changed
- **Layer window**: removed the up/down **reorder controls**; draw order fixed in code.
- **Initial map zoom 9 → 14** so projects (≥11) and labels (≥13) show on load.
- 🌎 Earth link tuned from angled 3D (`45t`, 800 m) to **top-down** (`0t`, 300 m).

### Infra
- New public repo `Wildhare1966/entitlement-reporter`; GitHub Pages from `main` root.
- Removed the duplicate `leadsdeals-reporter.html` from `indiana-mip-tracker` (single source of truth).
- Project scaffold: `CLAUDE.md`, `HANDOFF.md`, `README.md`, `.claude/launch.json`, `.gitignore`.

HEAD at end of S1: `2361c6e`.
