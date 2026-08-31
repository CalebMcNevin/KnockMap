# DOX framework

- DOX is highly performant AGENTS.md hierarchy installed here
- Agent must follow DOX instructions across any edits

## Project

- KnockMap: static single-page tool that maps Weed Man door-knocking season report CSVs by US zip / Canadian FSA onto a Leaflet map (choropleth boundaries, census housing denominators, per-metric color scales)
- Stack: one `index.html` of vanilla JS (Leaflet 1.9 + PapaParse via CDN), no framework, no backend, no build step; works over HTTP or `file://`
- UI must follow the Weed Man brand guide at `../STYLE_GUIDE.md` (tokens are CSS vars in `:root`; canvas colors hardcode the same hex values since canvas cannot use CSS vars)
- Generated data files, never hand-edit, regenerate with the build scripts:
  - `centroids.js` from `python3 build_centroids.py` (GeoNames); GeoNames' free CA/US files lack some codes (e.g. L3L, R5J..R5T, S7A..S7C, T6Y, V7Z); rows for centroid-less codes draw via `BOUNDARIES` (circle fallback skipped), and draw-time "not found" counts only rows with neither boundary nor centroid
  - `boundaries.js` (zip/FSA polygons with holes; polygon = `[outer ring, hole rings...]` in `[lat,lng]`, ~11MB) from `python3 build_boundaries.py` (see its header for the full download + mapshaper pipeline). Holes are load-bearing: L4H wraps around Kleinburg (L0J), and Leaflet cuts holes from multi-ring polygons
  - `census_homes.js` (occupied housing counts) from `python3 build_census_homes.py`
- `build_census_homes.py` reads `US_CENSUS_API_KEY` from gitignored `.env`; never print, echo, or commit that key
- Distribution zip `DoorKnockingMap.zip` is a build artifact, gitignored; rebuild with `zip -9 DoorKnockingMap.zip index.html centroids.js boundaries.js census_homes.js README.txt example.csv door-knocking.png zip-fsa-report.png report-settings.png` (name the three screenshots explicitly; `*.png` sweeps in debug screenshots)
- Dev server: `python3 -m http.server 8901` from the repo root
- Map tiles come from Esri World Light Gray Canvas (no key, no Referer requirement); do not switch back to OSM tiles (blocks `file://`) or CARTO (now requires an API key)

## Core Contract

- AGENTS.md files are binding work contracts for their subtrees
- Work products, source materials, instructions, records, assets, and durable docs must stay understandable from the nearest applicable AGENTS.md plus every parent AGENTS.md above it

## Read Before Editing

1. Read the root AGENTS.md
2. Identify every file or folder you expect to touch
3. Walk from the repository root to each target path
4. Read every AGENTS.md found along each route
5. If a parent AGENTS.md lists a child AGENTS.md whose scope contains the path, read that child and continue from there
6. Use the nearest AGENTS.md as the local contract and parent docs for repo-wide rules
7. If docs conflict, the closer doc controls local work details, but no child doc may weaken DOX

Do not rely on memory. Re-read the applicable DOX chain in the current session before editing.

## Update After Editing

Every meaningful change requires a DOX pass before the task is done.

Update the closest owning AGENTS.md when a change affects:

- purpose, scope, ownership, or responsibilities
- durable structure, contracts, workflows, or operating rules
- required inputs, outputs, permissions, constraints, side effects, or artifacts
- user preferences about behavior, communication, process, organization, or quality
- AGENTS.md creation, deletion, move, rename, or index contents

Update parent docs when parent-level structure, ownership, workflow, or child index changes. Update child docs when parent changes alter local rules. Remove stale or contradictory text immediately. Small edits that do not change behavior or contracts may leave docs unchanged, but the DOX pass still must happen.

## Hierarchy

- Root AGENTS.md is the DOX rail: project-wide instructions, global preferences, durable workflow rules, and the top-level Child DOX Index
- Child AGENTS.md files own domain-specific instructions and their own Child DOX Index
- Each parent explains what its direct children cover and what stays owned by the parent
- The closer a doc is to the work, the more specific and practical it must be

## Child Doc Shape

- Create a child AGENTS.md when a folder becomes a durable boundary with its own purpose, rules, responsibilities, workflow, materials, or quality standards
- Work Guidance must reflect the current standards of the project or user instructions; if there are no specific standards or instructions yet, leave it empty
- Verification must reflect an existing check; if no verification framework exists yet, leave it empty and update it when one exists

Default section order:
- Purpose
- Ownership
- Local Contracts
- Work Guidance
- Verification
- Child DOX Index

## Style

- Keep docs concise, current, and operational
- Document stable contracts, not diary entries
- Put broad rules in parent docs and concrete details in child docs
- Prefer direct bullets with explicit names
- Do not duplicate rules across many files unless each scope needs a local version
- Delete stale notes instead of explaining history
- Trim obvious statements, repeated rules, misplaced detail, and warnings for risks that no longer exist

## Closeout

1. Re-check changed paths against the DOX chain
2. Update nearest owning docs and any affected parents or children
3. Refresh every affected Child DOX Index
4. Remove stale or contradictory text
5. Run existing verification when relevant
6. Report any docs intentionally left unchanged and why

## User Preferences

- Commit only when the user explicitly asks
- No long dashes (en/em) in user-visible strings; use words ("to", ":", "n/a")
- Column display conventions: `%` suffix for names matching /pct|percent|rate/ plus `RATE_COLS` (derived conversion rates named without "rate"), `$` prefix for /amount|revenue/ (never /sale/); `DISPLAY` renames columns at the UI layer only (HomeCount shows as Homes, BadCount as Bad Leads) — internal CSV names feed every derivation, never rename those
- Metric dropdown and popup share one classifier (`classifyCol`); keep both grouped identically
- CityList and ZipOrFSA are excluded from popups; CityList values can be enormous and dirty (clamp behind a toggle); `HIDDEN_DISPLAY` (currently NoCount, AvgRevenuePer100Homes) is parsed but kept out of dropdown and popups; `HIDDEN_DROPDOWN` (currently AreaKm2) stays in popups but not the metric dropdown
- Derived metrics live in two places: CSV-derived (per-lead, per-contact, RevenuePer100Knocks) in `loadRows`, geometry-derived (AreaKm2, HomesPerKm2, HomesPerSqMile, per-km² outcomes) in `deriveDensity` which reruns when `boundaries.js` finishes lazy-loading

## Verification

- `node --check` on the extracted inline `<script>` (extraction one-liner lives in shell history of recent work; re-extract with python regex) before browser testing
- Serve with `python3 -m http.server 8901` and exercise in a browser (CDP via the `browser-use` harness; upload `example.csv` with `DOM.setFileInputFiles`, assert via `Runtime.evaluate` returning primitives only — returning DOM/Leaflet objects throws "Object reference chain is too long")

## Child DOX Index

- `data/AGENTS.md` — raw downloaded geo/census sources and extracted shapefiles (gitignored build inputs)
- Root-owned files: `index.html`, `build_centroids.py`, `build_boundaries.py`, `build_census_homes.py`, generated `centroids.js`/`boundaries.js`/`census_homes.js`, `README.txt`, `example.csv`, `sample_report.csv`, the three WEMMS screenshots (`door-knocking.png`, `zip-fsa-report.png`, `report-settings.png`), `.gitignore`, `.env` (key only, never content)
