# Competitor Entitlement Reporter

Single-file web-GIS app (ArcGIS Maps SDK for JS 4.31, no build step) that maps the
`LeadsDeals_Arbor` hosted feature layer — status-colored entitlement polygons with a filterable
sidebar, record detail, and a toggleable reference/market/utility layer stack.

**Live:** https://wildhare1966.github.io/entitlement-reporter/

## Develop
```sh
python -m http.server 8080   # then open http://localhost:8080/
```
Everything is in [`index.html`](index.html). Config (service URL, views, fields, status colors,
extra layers) is in the `CONFIG` block near the top of the `<script>`.

## Deploy
Commit to `main` and push — GitHub Pages serves the repo root and rebuilds in ~30–90s.

See [HANDOFF.md](HANDOFF.md) for the full architecture brief and [CLAUDE.md](CLAUDE.md) for
agent-facing notes (verified field schema, behavior contracts, gotchas).
