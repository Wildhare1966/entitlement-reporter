# NEXT CHAT KICKOFF — Competitor Entitlement Reporter

> Hand-off into the next session. Read this + [CLAUDE.md](CLAUDE.md) first. Full architecture brief =
> [HANDOFF.md](HANDOFF.md); change history = [CHANGELOG.md](CHANGELOG.md).
> **Last session: S1 (Jun 29 2026).** Next session = S2.

## Project in one line
Standalone single-file ArcGIS Maps SDK JS 4.31 web app (`index.html`, no build) that maps the hosted
**`LeadsDeals_Arbor`** feature layer as status-colored entitlement polygons, with a filterable sidebar,
record detail panel, toggleable reference/market/utility layers, and per-record Google deep-links.

## Live coordinates (verify before trusting)
| | |
|---|---|
| Repo | `Wildhare1966/entitlement-reporter` (public) |
| Live URL | **https://wildhare1966.github.io/entitlement-reporter/** |
| Pages source | `main` branch, root (`/`) |
| HEAD at handoff | **`2361c6e`** |
| Data layer | `https://services.arcgis.com/sW49gHAAsKKspEo8/arcgis/rest/services/LeadsDeals_Arbor/FeatureServer/0` (owned upstream by MIP ArcGIS sync) |
| Local preview | `cd entitlement-reporter && python -m http.server 8080` → http://localhost:8080/ |

## What S1 did (all LIVE)
1. **Field-name fixes** against the verified live schema: detail/card fields were referencing names that
   don't exist → fixed `Description→Terms`, `Owner_Name→OwnerName`, `Broker_Contact→Broker`. Added a
   fuzzy fallback to `rf()` for future schema drift.
2. **Date bug:** `Hearing_Date` is type **date-only** (was only caught for type `date`) → now formatted,
   and `fmtDate()` uses **UTC getters** to avoid the off-by-one day.
3. **BLC LTM theme:** `Sale_Price` doesn't exist → graduated color now on `Close_Price`.
4. **Layer window:** removed the up/down **reorder controls** (draw order fixed in code:
   Tracked Projects on top, reference layers beneath).
5. **Map opens at `zoom:14`** (was 9) so projects (visible ≥11) + labels (≥13) show immediately.
6. **Hosting:** spun this out of `indiana-mip-tracker` into its **own repo + GitHub Pages**; removed the
   old `indiana-mip-tracker/leadsdeals-reporter.html` copy (single source of truth).
7. **Google views (deep-link, no API key, ToS-clean)** — per-record buttons in the detail panel:
   - **🛰 Google Maps** — satellite, zoom 18, at the parcel centroid.
   - **🌎 Earth** — top-down (0° tilt) ~300 m above the site.
   - **👁 Street View** — *click-to-place*: enters a pick mode (blue hint banner + crosshair), user
     clicks a road on the map → pano opens there (centroid is inside the lot, has no panorama).
     Esc / Cancel exits.

## ⚠ Must-verify on a networked machine (S1 sandbox couldn't reach `services.arcgis.com`)
The code paths were structurally verified (no console errors, URL formats correct), but the live
data-dependent flows were **not** exercised. In a browser with network, confirm:
- [ ] Polygons render; the 3 view buttons (Weekly / Competitor Proposed / All) list records.
- [ ] Detail panel **Terms / Owner Name / Broker Contact** populate (not blank).
- [ ] **Hearing Date** shows `M/D/YY`, correct day.
- [ ] **🌎 Earth** opens straight-down ~300 m over the parcel.
- [ ] **👁 Street View** → click a nearby road → lands in the panorama at that spot.
- [ ] BLC LTM layer (toggle on) graduates color by `Close_Price`.

## Open / candidate items for S2+
- **External `EXTRA_LAYERS` CORS** is untested in-browser (`gisdata.in.gov` parcels/schooldist/flood/
  wetlands/contours, the `*.img.arcgis.com` DEM ImageServer, IURC electric view). A failed layer flags
  its row red ("unavailable"); fix = rehost + swap the `url` in CONFIG.
- **Thematic field names** (verified S1): Zonda `Annual_Starts` ✓, BLC `Close_Price` ✓ (fixed),
  utilities `name`/`utilityname` resolve via fuzzy `resolveField`. The other reference layers weren't
  field-checked (they're categorical/imagery, lower risk).
- **Optional product asks not yet built** (from the Google-views menu):
  - Surface the Google buttons on **list cards** too (one-click without opening detail).
  - **Esri native 3D toggle** (SceneView) — free, no Google key. Good "3D imagery" path if wanted.
  - **Embedded** Google Street View panel / Photorealistic 3D Tiles — needs a Google Maps API key +
    billing (and 3D Tiles realistically means a separate Cesium viewer).
- **Housekeeping (in the *other* repo, not this one):** `indiana-agenda-tracker/.claude/launch.json` has
  stale preview entries `leadsdeals-reporter` (:8760→Downloads) and `entitlement-reporter` (:8761) — the
  :8761 one is handy, :8760 is obsolete. Superseded Downloads copies
  (`leadsdeals-reporter (6).html`, `leadsdeals-reporter-patched.html`) can be deleted.

## Deploy recipe (this folder IS the repo — no clone dance)
```sh
cd C:\Users\ty\entitlement-reporter
# edit index.html ...
git add index.html
git commit -m "..."
git push origin main          # GitHub Pages rebuilds in ~30–90s
# verify: curl -s https://wildhare1966.github.io/entitlement-reporter/ | grep <new-marker>
```

## Gotchas (also in CLAUDE.md)
- Schema is **owned upstream** by `indiana-agenda-tracker/arcgis/sync_tracked_projects.py`; if a field
  vanishes, check that sync. Verified field list is in CLAUDE.md.
- `Hearing_Date` = date-only → format with UTC getters.
- This repo has its **own `.git`**, nested under `C:\Users\ty`; the home-rooted MIP repo's `.gitignore`
  denies it by default, so the two never entangle. Keep it that way.
- Google deep-links use **official URL forms** (Maps URL API + Earth web URL) — no key, ToS-clean.
  Embedding Google imagery *inside* the map would require a key + billing.
