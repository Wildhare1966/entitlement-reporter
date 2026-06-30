# NEXT CHAT KICKOFF — Competitor Entitlement Reporter

> Hand-off into the next session. Read this + [CLAUDE.md](CLAUDE.md) first. Full architecture brief =
> [HANDOFF.md](HANDOFF.md); change history = [CHANGELOG.md](CHANGELOG.md).
> **Last session: S2 (Jun 30 2026).** Next session = S3.

## Project in one line
Standalone single-file ArcGIS Maps SDK JS 4.31 web app (`index.html`, no build) that maps the hosted
**`LeadsDeals_Arbor`** feature layer as status-colored entitlement polygons, with a filterable sidebar,
record detail panel, toggleable reference/market/utility layers, per-record Google deep-links, a
**Google-Earth KML export**, and a **2D/3D toggle**.

## Live coordinates (verify before trusting)
| | |
|---|---|
| Repo | `Wildhare1966/entitlement-reporter` (public) |
| Live URL | **https://wildhare1966.github.io/entitlement-reporter/** |
| Pages source | `main` branch, root (`/`) |
| HEAD at handoff | **latest on `main`** — run `git log -1` (the S2 commit) |
| Data layer | `https://services.arcgis.com/sW49gHAAsKKspEo8/arcgis/rest/services/LeadsDeals_Arbor/FeatureServer/0` (owned upstream by MIP ArcGIS sync) |
| Local preview | `cd entitlement-reporter && python -m http.server 8080` → http://localhost:8080/ |

## What S2 did (all committed; verify live in a real browser)
1. **🌎 Earth KML export** — new rail button (`#kmlbtn`). Exports all 4-status tracked projects to a
   styled `LeadsDeals_Arbor.kml` matching the app (per-status `<Style>` from `STATUS_SYMBOLS`,
   `DETAIL_FIELDS` table per placemark, UTC `Hearing_Date`, WGS84, multipart polygons). **Verified live
   against the FeatureServer: 452 placemarks, 4 styles, 646 polygons.**
2. **2D/3D toggle** — new rail button (`#view3dbtn`, ⛰ 3D ⇄ 🗺 2D). Swaps `MapView`↔`SceneView` on the
   same map/layers, preserves the camera, re-wires the click handler. **Free, no API key.** Tracked
   Projects extrude into **status-colored 3D blocks** in the globe (`build3DRenderer`, 120 m).
   *Verified by DOM/console (canvas, 3D widgets, clean round-trip, record-click works in 3D); the WebGL
   globe itself could not be screenshotted in the sandbox — eyeball it in a real browser.*
3. **Google Earth "follow"** — the detail-panel 🌎 Google Earth button reuses one tab (`ge_arbor`) and
   re-points it to each selected record instead of piling up windows.
4. **Detail buttons** — removed 🛰 Google Maps; renamed Earth → **Google Earth**; Earth camera `300d →
   4525d` (top-down) to frame the whole site.
5. **Bug fix** — `view.destroy()` was destroying the shared `map`; now detach (`view.map=null`) first.

## ⚠ Must-verify in a real browser (sandbox can reach ArcGIS but can't render/screenshot WebGL)
- [ ] **3D toggle** actually shows the globe + extruded colored blocks; 2D↔3D keeps your view position.
- [ ] **🌎 Earth KML** downloads, and the file opens in Google Earth with correct status colors + field
      balloons. (Operator confirmed S2: file opens; this is the styling/field spot-check.)
- [ ] **🌎 Google Earth** button opens ONE tab and re-points it as you click through list records.
- [ ] Carry-over from S1 (data-dependent, still worth a glance): detail **Terms / Owner Name / Broker**
      populate; **Hearing Date** shows correct day; BLC LTM graduates by `Close_Price`.

## Open / candidate items for S3+
- **3D polish:** block height is a flat 120 m (`build3DRenderer`). Could make it **data-driven**
  (scale by `Lots` or `Acres`) so bigger projects read taller. Edges/opacity also tunable there.
- **Google photorealistic 3D** (the "real Google Earth look" inside our UI): needs a Google Maps API
  key + billing + a real rewrite (re-query FeatureServer, redraw layers on `Map3DElement`/Cesium).
  Only if the Esri globe isn't enough. Researched in S2 — see CHANGELOG.
- **Live KML sync into Google Earth** (declined in S2, here for the record): the genuinely hands-off,
  stays-current path is **Earth Pro + a hosted KML URL + NetworkLink** (commit KML to the repo so Pages
  serves it; add a small NetworkLink). Earth **Web** can't auto-load a URL or refresh. Direct Drive
  upload is blocked by the connector's inline-size limit (~170k tokens for 452 polygons).
- **Surface Google buttons on list cards** (one-click without opening detail) — still unbuilt from S1.
- **External `EXTRA_LAYERS` CORS** still untested in-browser (gisdata.in.gov parcels/schooldist/flood/
  wetlands/contours, the `*.img.arcgis.com` DEM ImageServer, IURC electric). Failed row flags red;
  fix = rehost + swap the `url`.

## Deploy recipe (this folder IS the repo — no clone dance)
```sh
cd C:\Users\ty\entitlement-reporter
# edit index.html ...
git add -A
git commit -m "..."
git push origin main          # GitHub Pages rebuilds in ~30–90s
# verify: curl -s https://wildhare1966.github.io/entitlement-reporter/ | grep <new-marker>
```

## Gotchas (also in CLAUDE.md)
- Schema is **owned upstream** by `indiana-agenda-tracker/arcgis/sync_tracked_projects.py`; if a field
  vanishes, check that sync. Verified field list is in CLAUDE.md.
- `Hearing_Date` = date-only → format with UTC getters.
- **`view.destroy()` destroys the shared `map`** unless you `view.map=null` first (S2 bug, now guarded
  in `buildView`). Keep that if you touch the view lifecycle.
- A web page can only drive a window **it** opened (the `ge_arbor` Earth tab) — it can't steer a
  separately-opened Earth, load a local file into Earth, or remote-control Earth Pro. Browser sandbox.
- This repo has its **own `.git`**, nested under `C:\Users\ty`; the home-rooted MIP repo's `.gitignore`
  denies it by default, so the two never entangle. Keep it that way.
- Google deep-links use **official URL forms** (Earth web URL) — no key, ToS-clean. Embedding Google
  imagery *inside* the map would require a key + billing.
