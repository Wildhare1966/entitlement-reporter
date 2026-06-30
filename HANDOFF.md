# Claude Code Handoff — `leadsdeals-reporter.html`

> Purpose: hand this single-file app to Claude Code for analysis and further revision. This doc is a code map + architecture brief + known-issues list + test protocol. Read the file alongside this. ~823 lines, single self-contained HTML.

---

## 0. What this is (one paragraph)
A single-file web GIS app (no build step) built on the **ArcGIS Maps SDK for JavaScript 4.31 via CDN**. It replaces an ArcGIS Experience Builder layout. It renders a public hosted feature layer (`LeadsDeals_Arbor`) as status-colored polygons on a map, with a right-side sidebar offering three filtered list "views," a record detail panel, a reorderable multi-layer widget, basemap switcher, and a settings panel. Target host: GitHub Pages (user: wildhare1966). No framework, no bundler — vanilla JS inside one `require([...])` AMD callback. Theme: blue, dark navy panels. CSS uses custom properties in `:root`.

---

## 1. Run / test locally
- It must be served over HTTP (the SDK + CORS need it), not opened as `file://` for full function. Quick: `python3 -m http.server 8000` then open `http://localhost:8000/leadsdeals-reporter.html`.
- No npm install. The only dependency is the CDN `<script src="https://js.arcgis.com/4.31/">` and its CSS `<link>` in `<head>`.
- The build environment used to author this could NOT reach `services.arcgis.com` or `gisdata.in.gov`, so live data behavior was never executed here. First real test happens in the browser.

---

## 2. File structure (line numbers approximate, will drift after edits)
- **1–10** head: title, ArcGIS dark theme CSS link, SDK CDN script.
- **11–~210** `<style>` — all CSS. Custom props in `:root` (~12–28). Key blocks: `#buttonbar` (floating view buttons), `.viewbtn`, `#railbar`/`.railtool`, `.card*` (list cards), `.popover` (filter/settings), `#detail*` (feature info), `#layerbtn`/`#basemapbtn`/`#layerpanel`/`#basemappanel`, `.lyr`/`.reorder` (reorderable rows), `.field input[type=range]` (settings sliders).
- **~213–310** `<body>` markup: `#map` (holds floating buttons + basemap/layer panels), `#sidebar` (railbar -> listhead -> #list, plus filter/settings popovers), `#detail` overlay.
- **318–386 CONFIG BLOCK** (edit most things here — see §3).
- **388 onward** single `require([...])` callback with all logic.

### Function index (inside the require callback)
| Line | Function | Role |
|---|---|---|
| 421 | `buildRenderer()` | unique-value renderer on Status; `defaultSymbol:null` so unlisted statuses don't draw |
| 438 | `buildLabels()` | two label classes: LeadsDeals (yellow/black halo) + Builder (black/bright halo); both gated by `settings.labelZoom` via minScale |
| 477 | `applyZoomVisibility()` | sets `leadsLayer.minScale/maxScale` from `settings.minZoom/maxZoom`; rebuilds labels |
| 487 | `preloadCounts()` | per-view `queryFeatureCount` -> button count badges |
| 500 | `buildLayerPanel()` | renders the reorderable layer list from `layerOrder[]` |
| 526 | `moveLayer(idx,dir)` | swaps entries in `layerOrder[]`, re-applies map order, re-renders panel |
| 533 | `applyMapOrder()` | `map.reorder()` each layer so list-top = draw-top; pins flash/select layers on top |
| 545 | `themeColor(i)` | categorical palette for `applyTheme` |
| 552 | `applyTheme()` | categorical unique-value renderer (utilities, by NAME/Utility_Name) |
| 570 | `resolveField(layer,wanted)` | **critical**: case-insensitive + fuzzy (strip non-alphanumerics, then includes-match) field-name resolver for thematic layers |
| 578 | `fieldStats(layer,field)` | server-side min/max via outStatistics |
| 589 | `applySizeTheme()` | graduated **size** visualVariable (Zonda by Annual_Starts) |
| 608 | `applyColorTheme()` | graduated **color** visualVariable (BLC by Sale_Price), 5-stop high->low red ramp |
| 635 | `toggleExtra(cfg,on,row)` | lazily creates FeatureLayer/ImageryLayer on first toggle; applies the right thematic; red-flags row on load failure |
| 659 | `selectView(key)` | switches the three views; shows/hides filter button + Status filter field; sets baseWhere |
| 673 | `whereClause()` | combines baseWhere AND extraWhere (from filters) |
| 675 | `loadList()` | queries filtered+sorted features, renders cards |
| 695 | `makeCard(feature)` | 3-row list card (no summary link here by design) |
| 713 | `openDetail(feature)` | builds the detail panel field rows; zoom-to + highlightSelected + bloomFlash |
| 741 | `highlightSelected(geom)` | persistent neon-teal outline on `selectLayer` |
| 749 | `bloomFlash(geom)` | 3x pulse on `flashLayer` then clear |
| 780 | `populateFilterOptions(key)` | distinct-value queries to fill filter dropdowns |

### S2 additions (current line numbers; `index.html` is now ~1028 lines)
| Line | Function | Role |
|---|---|---|
| ~451 | `wireView()` | re-attaches `onMapClick` + cursor to the live view after a 2D/3D rebuild |
| ~455 | `buildView(use3D)` | **2D/3D toggle core**: destroys old view (after `view.map=null` so the shared map survives), creates `MapView` or `SceneView` on the same `#map`, restores `viewpoint`, re-wires, swaps the Tracked-Projects renderer (2D fills ↔ 3D extrude), updates the `#view3dbtn` label |
| ~494 | `build3DRenderer()` | unique-value `polygon-3d`/`extrude` renderer (status colors, 120 m blocks) for the SceneView; counterpart to `buildRenderer()` |
| ~816 | `earthUrl(lat,lng)` | builds the top-down Google Earth web URL (`0a,4525d,…,0t`) |
| ~819 | `openEarth(lat,lng)` | user-gesture: open/reuse the single `ge_arbor` Earth tab and re-point + focus it |
| ~826 | `followEarth(lat,lng)` | passive: re-point an already-open `ge_arbor` tab on record select (no focus steal, no new tab) |
| ~829 | `setGeoLinks(geom)` | builds the detail-panel Google buttons (🌎 Google Earth, 👁 Street View); calls `followEarth` |
| ~860 | `kmlColor(rgba)` | `[r,g,b,a]` → KML `aabbggrr` hex |
| ~868 | `geomToKml(geom)` | Esri rings → KML Polygon/MultiGeometry; CW ring = outer, CCW = hole (`ringIsCW`) |
| ~885 | `descTable(a)` | per-placemark `<description>` field table from `DETAIL_FIELDS` (UTC dates, `LINK_FIELDS` links) |
| ~892 | `buildKml(features)` | assembles the full KML doc: one `<Style>` per status + a `<Placemark>` each |
| ~910 | `exportKML()` | `#kmlbtn` handler: queries all-status features (geom, `outSR 4326`) → `buildKml` → downloads `LeadsDeals_Arbor.kml` |
| ~958 | `onMapClick(e)` | hoisted map-click handler (Street-View pick OR hitTest→openDetail); attached by `wireView` |

**New rail buttons** (`#railbar`): `#kmlbtn` (🌎 Earth KML export) and `#view3dbtn` (⛰ 3D ⇄ 🗺 2D).
**New module state:** `is3D` (bool), `geWin` (the reused Google Earth tab handle).

### Module-level state (top of require callback, ~395–405)
`fieldMap` (lowercase->actual field name), `dateFields` (Set), `currentView`, `selectedOID`, `baseWhere`, `extraWhere`, `leadsLayer`, `view`, `flashLayer`, `selectLayer`, `layerObjs` (id->layer), `layerOrder` (ordered array driving the panel + draw order), `settings` (`{labels, minZoom:11, labelZoom:13, maxZoom:24}`).

---

## 3. CONFIG block — where to change things (lines 318–386)
- `SERVICE_URL` (320) — the LeadsDeals layer endpoint.
- `VIEWS` (323) — the three buttons: each has `where`, `filter` (bool, shows filter button), `status` (bool, shows Status filter field). Edit here to change filters.
- `DETAIL_FIELDS` (333) — `[realFieldName, displayAlias]` pairs, in panel order. "Description"->"Terms" alias lives here.
- `LINK_FIELDS` (just below) — fields rendered as "View" links: currently `Summary, Recent_Agenda, FolderLink`.
- `CARD` (342) — role->field mapping for list card (name/juris/builder/hearing/date/terms).
- `STATUS_SYMBOLS` (346) — the four-status color/border/style map. Edit colors here.
- `EXTRA_LAYERS` (354) — the 11 toggleable layers. Each: `{group, id, title, type:"feature"|"image", url, opacity?, theme?|sizeTheme?|colorTheme?}`. To add a layer, push an object here; the panel rebuilds from `layerOrder` which is derived from this array at 497-ish.

---

## 4. Behavior contracts (don't break these in revision)
- **Map shows ONLY four statuses.** Enforced by `leadsLayer.definitionExpression` set right after schema load (search "definitionExpression"). The renderer's `defaultSymbol:null` is the backstop.
- **Sidebar starts collapsed**; a view button must be clicked first. Buttons float over the map so they're always reachable.
- **List sort:** always Hearing Date DESC (newest/future first). `loadList()` orderByFields.
- **Record selection** (card or map polygon) must: open detail, zoom to polygon, draw persistent teal outline, flash 3x. Closing detail clears the teal outline.
- **Summary** is a "View" link in the DETAIL panel only — NOT on list cards (was removed intentionally).
- **Layer reorder:** top of panel list = top of map draw order. flash + select layers always stay above everything.
- **Tracked Projects** (base layer) is checked/visible on load.
- **(S2) 2D/3D toggle** must reuse the *same* `map` — `buildView` sets `view.map=null` before
  `view.destroy()` (else the SDK destroys the shared map: "The provided map is already destroyed").
  Camera position carries across via `viewpoint`; the click handler is re-attached via `wireView`.
- **(S2) Renderer parity:** 2D uses `buildRenderer` (simple-fill), 3D uses `build3DRenderer` (extrude);
  both are unique-value on `Status` and **must keep `STATUS_SYMBOLS` as the single color source**.
- **(S2) KML export** mirrors the app exactly — status `<Style>` from `STATUS_SYMBOLS`, fields from
  `DETAIL_FIELDS`, UTC `Hearing_Date`, `LINK_FIELDS` as links. If those configs change, the KML follows.
- **(S2) Google Earth button** reuses one tab (`ge_arbor`) and follows list selection; a web page can
  only drive a window it opened (can't load a local KML into Earth or steer a separate Earth/Earth Pro).

---

## 5. Known fragilities / likely revision targets
1. **Field names are guessed.** Config uses underscore names (`Last_Result`, `Recent_Hearing`, `Hearing_Date`, `Owner_Name`, `Broker_Contact`, `Recent_Agenda`, `FolderLink`, `Summary`, `Weekly`, `Status`). `rf()` (the resolver, ~411) lowercases and maps, but does NOT fuzzy-match the way `resolveField()` does. **If detail/card/label fields are blank, fix the names in CONFIG or upgrade `rf()` to use the fuzzy `resolveField` logic against `leadsLayer`.** This is the #1 likely task.
2. **Thematic field names guessed:** Zonda `Annual_Starts`, BLC `Sale_Price`, utilities `NAME`/`Utility_Name`. `resolveField()` fuzzy-matches, but a truly different name yields uniform symbols. Verify against each layer's REST `?f=json` field list.
3. **External layer reachability untested.** The 11 EXTRA_LAYERS include `gisdata.in.gov` services + an ImageServer + the IURC electric view. CORS or access failures flag the row red ("unavailable (use Drive)"). Failures move to Google Drive hosting; swap the `url`. (Esri ImageServer + state ArcGIS Server CORS are the usual suspects.)
4. **Zoom<->scale conversion is approximate.** `applyZoomVisibility` and `buildLabels` use `591657527 / 2^(z-1)` as a zoom->scale heuristic. Fine for whole-zoom thresholds; not exact at fractional zooms.
5. **`map.reorder` indices.** `applyMapOrder()` inverts list order to draw index. If layers are added/removed dynamically the inversion math (`(n-1)-i`) assumes only the tracked set is in `map.layers` plus flash/select. Adding non-tracked operational layers elsewhere could shift indices — keep additions going through `layerOrder`.
6. **Date filters use `DATE '...'` SQL.** Hosted feature service must accept standardized SQL date literals (it should). If date range filters error, check the field is a true date type vs. string.

---

## 6. Suggested verification protocol (Claude Code can script the queries)
For each of these, hit the REST endpoint `?f=json` and inspect `fields[]`:
1. LeadsDeals layer: confirm exact names for the 14 detail fields + Weekly + Status. Patch CONFIG or `rf()`.
2. Zonda: confirm the Annual Starts numeric field name.
3. BLC LTM: confirm the Sale Price numeric field name + that geometry is point.
4. Utilities 0/1/2 + IURC electric: confirm the category field name (NAME vs Utility_Name vs other).
5. For each EXTRA_LAYER url: a `query?where=1=1&returnCountOnly=true&f=json` smoke test for reachability/CORS.
Then in-browser: walk the 4 behavior contracts in §4.

---

## 7. Open product decisions (non-code)
- **Duplicate-status spec:** original brief listed "Competitor Active" twice; second (teal/dashed) was interpreted as **Competitor Proposed**. Confirm before relying on the teal symbology.
- **Elevation/ownership:** decide whether this stays a Ty-owned artifact or becomes team-regenerable, before the pattern scales to the weekly leadership Map Tour. A single-file custom app that only one person can edit cuts against the stated Elevation strategy; weigh that against EXB's configurability-for-all.

---

## 8. First-session prompt for Claude Code (paste to start)
> Analyze `leadsdeals-reporter.html` (single-file ArcGIS Maps SDK for JS app). First, fetch the REST schema for the LeadsDeals layer (`https://services.arcgis.com/sW49gHAAsKKspEo8/arcgis/rest/services/LeadsDeals_Arbor/FeatureServer/0?f=json`) and every URL in the EXTRA_LAYERS config block, and produce a table of (config field name -> actual field name) mismatches plus a reachability check per layer. Then patch the CONFIG block (lines ~318–386) and the `rf()` resolver so all detail/card/label/thematic fields resolve correctly. Preserve the behavior contracts in §4 of the handoff doc. Don't deploy; show me the diff first.
