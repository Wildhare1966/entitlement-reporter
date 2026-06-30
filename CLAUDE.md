# CLAUDE.md — Competitor Entitlement Reporter

> Auto-loaded every session. Standalone single-file web-GIS app. Keep this short and stable.

## What this is
A **single self-contained HTML file** (`index.html`, ~825 lines, no build step) built on the
**ArcGIS Maps SDK for JavaScript 4.31 via CDN**. It renders the hosted feature layer
**`LeadsDeals_Arbor`** as status-colored polygons on a map, with a right-side sidebar offering three
filtered list "views" (Weekly / Competitor Proposed / All), a record detail panel, a toggleable
multi-layer widget (parcels, school districts, market, environmental, terrain, utilities), a basemap
switcher, and a settings panel. Vanilla JS inside one `require([...])` AMD callback. Dark navy / blue
theme via CSS custom properties in `:root`. Operator: Tyrone (`Wildhare1966`).

Read first: **[NEXT_CHAT_KICKOFF.md](NEXT_CHAT_KICKOFF.md)** (current state + open items), then
**[HANDOFF.md](HANDOFF.md)** (full code map, function index, behavior contracts).
Change history: **[CHANGELOG.md](CHANGELOG.md)**. Live: https://wildhare1966.github.io/entitlement-reporter/

## Data source (owned upstream — do not assume you control the schema)
- Layer: `https://services.arcgis.com/sW49gHAAsKKspEo8/arcgis/rest/services/LeadsDeals_Arbor/FeatureServer/0`
- This layer is populated by the **MIP tracker's one-way ArcGIS sync**
  (`indiana-agenda-tracker/arcgis/sync_tracked_projects.py`). If a field is missing or renamed, the fix
  usually belongs to CONFIG here, but the field *catalog* lives in that other repo.
- **Verified live schema (Jun 2026)** — exact field names this app depends on:
  `LeadsDeals, FolderLink, Status, Acres, Acq_Mngr, School_District, Lot_Size, Terms, Jurisdiction,
  Last_Result, OwnerName, Builder, GlobalID, Purchase_Type, LeadID, Broker, Lots (double),
  Hearing_Date (DATE-ONLY), Summary, Recent_Agenda, Recent_Hearing, weekly, Code`.
  Object-id field = `FID`. ⚠ Gotchas baked into the code: `Hearing_Date` is type **date-only** (format
  with UTC getters, else it rolls back a day); `OwnerName`/`Broker`/`Terms` are the real names (NOT
  `Owner_Name`/`Broker_Contact`/`Description`).
- Status values in use: `Competitor Proposed, Competitor Active, Arbor Active, Under Contract`
  (the map's `definitionExpression` shows only these four). `weekly` ∈ {`yes`,`no`}.

## Where to change things — CONFIG block (top of the `<script>`, ~line 318–386)
`SERVICE_URL`, `VIEWS` (the 3 buttons), `DETAIL_FIELDS` (`[realField, alias]`), `LINK_FIELDS`,
`CARD` (list-card role→field), `STATUS_SYMBOLS` (4-status colors), `EXTRA_LAYERS` (toggleable layers).
`rf()` resolves config names to real field names case-insensitively, with a fuzzy fallback for drift.

## Behavior contracts (don't break in revision)
- Map shows ONLY the four statuses (via `definitionExpression` + `defaultSymbol:null`).
- Sidebar starts collapsed; a view button must be clicked first. List always sorts Hearing Date DESC.
- Record select (card or polygon) → open detail, zoom to polygon (zoom 14), persistent teal outline,
  3× flash. Closing detail clears the outline.
- Summary is a "View" link in the DETAIL panel only, never on list cards.
- Layer reorder UI was **removed** (P-this-session); draw order is fixed: Tracked Projects on top,
  reference layers beneath. `applyMapOrder()` still enforces it.
- Map opens at `zoom:14` so projects (visible ≥11) and labels (≥13) show immediately.

## Run locally
- Must be served over HTTP (SDK + CORS), not `file://`.
- `.claude/launch.json` defines a `python -m http.server` config (port 8080). Or:
  `python -m http.server 8080` in this folder → `http://localhost:8080/`.

## Deploy (this IS the live site)
- Repo: **`Wildhare1966/entitlement-reporter`** (public). GitHub Pages serves `main` root.
- Live URL: **https://wildhare1966.github.io/entitlement-reporter/**
- Ship: edit `index.html` → `git commit` → `git push origin main`. Pages rebuilds in ~30–90s.
- No clone-the-deployed-repo dance — this folder **is** the repo (unlike the MIP tracker FE).

## Gotchas / fragilities
- External `EXTRA_LAYERS` (gisdata.in.gov, ImageServer, IURC) can fail CORS in-browser — the row turns
  red ("unavailable"). Fallback: rehost the layer and swap its `url`.
- Zoom↔scale is approximate (`591657527 / 2^(z-1)`); fine for whole-zoom thresholds.
- This repo's `.git` is its own, nested under `C:\Users\ty`. The home-rooted MIP repo's `.gitignore`
  denies everything by default, so this folder is invisible to it — keep it that way.
