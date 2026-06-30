# How to start the next chat (S3)

## 1. Open Claude Code in THIS folder
Open a new Claude Code session with the working directory set to:

```
C:\Users\ty\entitlement-reporter
```

- **Desktop app:** project/folder switcher → Open Project → pick `C:\Users\ty\entitlement-reporter`.
- **CLI:** `cd C:\Users\ty\entitlement-reporter` then `claude`.

This is its own repo (own `.git`) and auto-loads `CLAUDE.md`. Do **not** start from
`indiana-agenda-tracker` — that's the separate MIP project.

## 2. Paste this as your first message

> This is the **Competitor Entitlement Reporter** — a standalone single-file ArcGIS Maps SDK app
> (`index.html`, no build) live at https://wildhare1966.github.io/entitlement-reporter/ (repo
> `Wildhare1966/entitlement-reporter`, Pages from `main` root).
>
> Read `NEXT_CHAT_KICKOFF.md`, then `CLAUDE.md`, then skim the **S2** section of `CHANGELOG.md` for what
> the last session shipped. Run `git log --oneline -8` and `git status` to confirm state. Don't deploy
> or push until I confirm.
>
> Last session (S2) added: a **🌎 Earth KML export** button, a **2D/3D toggle** (Esri SceneView, with
> status-colored extruded blocks), a **Google Earth "follow one tab"** behavior, and detail-button
> cleanup. All committed. The sandbox can reach `services.arcgis.com` but can't render/screenshot WebGL,
> so the **3D globe was verified by DOM/console, not visually** — I may ask you about that.
>
> Then give me: (a) a 3-line status of where the project stands, and (b) the open-items list from the
> kickoff, so I can pick what to work on.

## 3. Likely first tasks (from the kickoff's open items)
- **3D polish:** make the extruded block height data-driven (`Lots`/`Acres`) in `build3DRenderer`.
- Eyeball-verify the S2 work in a real browser (3D globe + blocks; KML opens in Earth with right colors).
- Surface the Google buttons on list cards (one-click, carried over from S1).
- Decide on the Google *photorealistic* 3D path (needs API key + billing + rewrite) vs. staying on the
  free Esri globe.
- Check external `EXTRA_LAYERS` for CORS failures (rows flag red) and rehost any that fail.

## Deploy reminder
This folder **is** the repo. Edit `index.html` → `git add -A` → `git commit` → `git push origin main` →
Pages rebuilds in ~30–90s. No clone-the-deployed-repo step (unlike the MIP tracker FE).
