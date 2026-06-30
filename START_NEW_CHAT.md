# How to start the next chat (S2)

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
> (`index.html`) live at https://wildhare1966.github.io/entitlement-reporter/ (repo
> `Wildhare1966/entitlement-reporter`, Pages from `main` root).
>
> Read `NEXT_CHAT_KICKOFF.md`, then `CLAUDE.md`, then skim `CHANGELOG.md` for what the last session did.
> Run `git log --oneline -5` and `git status` to confirm the current state (HEAD should be `2361c6e` or
> later). Don't deploy anything until I confirm.
>
> Then give me: (a) a 3-line status of where the project stands, and (b) the open-items list from the
> kickoff, so I can pick what to work on. I especially want to know the results of the "must-verify on a
> networked machine" checklist — assume I'll run those manually unless you can reach the ArcGIS service.

## 3. Likely first tasks (from the kickoff's open items)
- Confirm the S1 "must-verify" checklist (detail fields populate, dates correct, Earth top-down,
  Street View click-to-place) — needs a browser with network to `services.arcgis.com`.
- Optional builds: Google buttons on list cards; Esri native 3D toggle (no key); embedded Google
  Street View / 3D Tiles (needs a Google Maps API key + billing).
- Check external `EXTRA_LAYERS` for CORS failures (rows flag red) and rehost any that fail.

## Deploy reminder
This folder **is** the repo. Edit `index.html` → `git commit` → `git push origin main` → Pages rebuilds
in ~30–90s. No clone-the-deployed-repo step (unlike the MIP tracker FE).
