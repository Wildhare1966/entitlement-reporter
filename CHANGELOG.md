# Changelog — Competitor Entitlement Reporter

Standalone single-file ArcGIS app. Live: https://wildhare1966.github.io/entitlement-reporter/
Sessions are labeled S1, S2, … (the app's own dedicated sessions).

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
