# DOX framework

- DOX is highly performant AGENTS.md hierarchy installed here
- Agent must follow DOX instructions across any edits

## Project

- KnockMap: static single-page tool that maps Weed Man door-knocking season report CSVs by US zip / Canadian FSA onto a Leaflet map (choropleth boundaries, census housing denominators, per-metric color scales)
- Permit tracking: records covering one or many zips/FSAs (comma/space separated codes list; one jurisdiction's rule shared by all its codes; codes can be appended or removed by clicking map areas via the form's Pick on map mode, which arms click-to-toggle with popups suppressed, Esc/button ends it, ends on save and dialog close) stored in localStorage `dkm-permits` (separate from `dkm-prefs`). The form's unsaved state renders live on the map (`LIVE_REC` overlays stored records in `permitRec`/`permitIdOf`: fills, hover groups, popups, ink; cleared on dialog close so discards revert). The save-warn dialog is a modal (top layer), always above the non-modal permit form, and centers itself on the form's current position (clamped to the viewport) so it reads as part of the form flow, edited in the Permits dialog (Edit Permits button on the main header). The dialog is non-modal (`show()`, z-index above map panes) and draggable by its heading (clamped to the viewport; last position kept in sessionStorage `dkm-permit-pos` and restored clamped on reopen; window resizes re-clamp so it is never stranded off screen), so the map stays hoverable and clickable while editing; selecting a record strokes all its codes' geometries on the map in ink (`permitSel`, works in both views, cleared on dialog close); opening another record (list click, New record, popup Edit/Add) with a dirty form routes through the same unsaved-changes dialog as closing (Close button, X corner button, Esc; no backdrop in non-modal mode). Shown as a categorical map overlay via the header Results/Permits view toggle (state in `PERMIT_VIEW`, persisted as `pOn`); the Knocking/Door Hangers picker (`#permitAct`, persisted as `pAct`) is the permit legend's header select on the map, not a header control, so the header never reflows when the view flips; in permit view hovering (or popup-pinning) a code highlights every shape whose record claims it (`s.grp` union via `claimIds`, post-shape-loop pass in `draw`; a split-jurisdiction code lights BOTH rules' areas); a code claimed by more than one record (split jurisdiction) renders with a diagonal hatch fill in the status color (`permitSplit` + `hatch` build a cached `CanvasPattern` off the canvas renderer's private `_ctx`; strokes stay solid; legend swatch `i.hatch`, tooltip appends "multiple rules"; the form draft replaces the edited record, never double-counts); also a popup Permits section only when a record exists (Edit button reopens it; codes with no record get an Add permit requirement button in the popup's top-right corner that prefills the clicked code); the dialog highlights the selected record in the list and lights the Save button orange while the form differs from the stored record; shared between computers via the dialog's Save/Load buttons (JSON export/import; Load errors on unreadable files and on parseable files with no valid records), plus a separate Export CSV button (one row per record, codes joined with "; ", human-readable status labels, not importable); the saving/persistence note lives in a `?` tooltip (`.q.tipL`, opens left) beside Save/Load, and the jurisdiction caveat lives in the Zips / FSAs `?` tooltip; zip/FSA is a proxy for the legal jurisdiction, the jurisdiction field carries the real authority
- Customer penetration: optional customer list CSV (header needs SERVICEPOSTALZIP; CUSTOMERID dedupes customers per code) loaded via the header Customers button, either before or after the report; derives Customers (count) and Customer Penetration % (vs CensusHomes or WEMMS HomeCount, via the Penetration vs picker) per code; re-upload replaces counts. Hidden at launch: `CUSTOMERS_ENABLED = false` gates the Customers button, the Penetration vs picker, and the `dkm-customers` stash restore; parsing/derivation code stays intact, flip the flag to re-enable
- Banner (toast): full-width bar under the header, now an overlay (`.bnr` elements inside `#bannerClip`, a `pointer-events:none` clipper child of the header) so the map never shifts when one appears or dismisses; opens sliding DOWN from under the header, closes retracting back up (`visibility` transition, not `display` flips: the bar stays rendered so both directions animate reliably; the `.hidden` utility's `display:none` is overridden with `!important` for these elements). Toasts stack: every `banner()` call appends its own element (newer paints above older, slides down over them) with its own dismiss timer; nodes self-remove after the retract. Error variant is the default; `banner(msg, true)` gives the green success variant (used by session restore). While any toast is visible, `body.banner-on` slides the Leaflet zoom controls down 44px and back. Stacking: legend 1000 < toasts 1500 < permitDialog 1600; map popups clear the toasts via a lift right after `L.map()` (popup pane reparented to the map container, map pane transform mirrored on `move zoom viewreset`; the map pane's persistent transform seals child panes in a z-400 stacking context that no child z-index can escape, so pan 1550 alone does nothing)
- Session restore: last uploaded report (`dkm-report`) and customer list (`dkm-customers`) text is stashed in localStorage and re-parsed on reload (quota overflow or corruption is skipped gracefully); localStorage keys are dkm-prefs, dkm-permits, dkm-report, dkm-customers; `navigator.storage.persist()` is requested on load as a best-effort guard against eviction
- Stack: one `index.html` of vanilla JS (Leaflet 1.9 + PapaParse via CDN), no framework, no backend, no build step; works over HTTP or `file://`
- UI must follow the Weed Man brand guide at `../STYLE_GUIDE.md` (tokens are CSS vars in `:root`; canvas colors hardcode the same hex values since canvas cannot use CSS vars)
- Header layout: the metric select is the emphasized primary control; permit controls live in `#permGrp` on the header itself (View-labeled Results/Permits view toggle, Edit Permits button); remaining secondary options (color scale, log, filter, and the Data sources button) live in the Settings popover `#settingsPanel` (native Popover API via `popovertarget`; no `display` rule on the closed state, or the UA's `display:none` gets overridden and the panel never hides). No hide-zeros toggle: the knocks filter covers it (reports contain no zero-knock codes)
- Help tour: 8-step walkthrough (`TOUR` array; card is always centered, the spot is the cutout in the dark overlay via a 9999px box-shadow) auto-runs once after the first CSV upload unless `tourDone` is set in `dkm-prefs`; the compass button `#tourBtn` in the header replays it (the last step's cutout lands on it). Veil/spot/card are `popover="manual"` because showing an auto popover closes every other open popover that is not its DOM ancestor (the trio must coexist; Esc and veil clicks end the tour in JS). During the tour `tourStep` also flips `#settingsPanel` itself to `popover="manual"` (otherwise a click on the card light-dismisses the auto panel and desyncs the `tourPanel` flag; `tourEnd` restores `"auto"`). Only the Settings step opens `#settingsPanel` (animated via `@starting-style`): `t2` entries are `[selector, ms, action]`, and its chain has `settingsClickDemo` visibly press the gear to really open the panel (`tourShowPanel` restacks so the trio paints above it and must re-show the trio itself when called from a phase timer; any step change closes the panel, entry always starts it closed). The view-toggle step's chain presses the toggle's Permits half (`permitClickDemo`, the press does the real `setView(true)` flip; `tourFlipped`, restored by `tourEnd` only if the tour itself flipped it), then the spotlight moves onto the collapsed legend picker (`#permitAct`, nothing expanded yet), then `permitActDemo` presses it and expands a replica dropdown `#tourActDD` under it listing both options with the current one highlighted while the spotlight widens to the whole legend block (`#permitLegend`; native `select.showPicker()` needs live user activation and cannot be closed programmatically), then `permitKnocksDemo` picks Knocking for real (green pick flash, `#permitAct` value/`pAct`/map update; there is no `permitAct` variable, the select's value is the state) and the dropdown folds back up before the spotlight shrinks to the collapsed picker (the act returns `"defer"` and schedules its own `tourPlace`, so the spot holds wide until the fold finishes); the replica is removed on any step change and by `tourEnd`. The rule-tracker step (`t: "#permitsBtn"`, `t2: [["#permitsBtn", 1000], ["#permitDialog", 2000]]`) does the same with `editPermitsClickDemo` pressing Edit Permits to really open the form while the highlight holds on the button for another second before the `#permitDialog` phase moves the spotlight onto the form (`tourPlacePermitDlg` puts it beside the centered card with a 50px gap; `margin: 0` inline, else the UA dialog centering re-ignores `left`; `tourDlg` claimed so `tourEnd` closes it and any step change closes it, re-entering step 7 replays the press) (out-of-order throttled timers are skipped by the `tourT2 !== k` guard)
- Generated data files, never hand-edit, regenerate with the build scripts:
  - `centroids.js` from `python3 build_centroids.py` (GeoNames); GeoNames' free CA/US files lack some codes (e.g. L3L, R5J..R5T, S7A..S7C, T6Y, V7Z); rows for centroid-less codes draw via `BOUNDARIES` (circle fallback skipped), and draw-time "not found" counts only rows with neither boundary nor centroid; centroid circles render only after boundary loading settles (loaded without a polygon for the code, or load failed with a redraw from the onerror handler), so circles never flash before the lazy `boundaries.js` geometry pops in
  - `boundaries.js` (zip/FSA polygons with holes; polygon = `[outer ring, hole rings...]` in `[lat,lng]`, ~11MB) from `python3 build_boundaries.py` (see its header for the full download + mapshaper pipeline). Holes are load-bearing: L4H wraps around Kleinburg (L0J), and Leaflet cuts holes from multi-ring polygons
  - `census_homes.js` (lawncare-eligible occupied dwellings: US ACS B25032 owner+renter detached/1-attached/2-unit/mobile; CA StatCan 2021 profile char 42-45,48,49; apartments excluded) from `python3 build_census_homes.py`
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
- Summary dropdowns (`#summary details > div`, z-index 1560: above toasts 1500 and map popups 1550, below the permit dialog 1600): "duplicates kept first" lists dup codes with `×N` occurrences (`dupes` Map in `loadRows`, `#dupCount`); one "not found" widget lists draw-time codes lacking boundary and centroid first, then a blank line, then load-time skipped rows (no readable or shape-valid zip/FSA: blanks plus junk like "Route 7" or "0" that can never match anything, `/^\d{5}$|^[A-Z]\d[A-Z]$/` gate in `loadRows`; raw values quoted, `(empty)` for blanks, capped at 8 plus "and N more") (`noZip`/`noZipVals` feed `nfCount` in `draw`); "filtered out" widget text includes a `#fLink` link whose click opens `#settingsPanel` then runs a spotlight glide (`#hlSpot`, a `popover="manual"` cutout shown after the panel so the top layer stacks it above; a 10px dot at the mouse morphs into the `#fltRow` rectangle while gliding, starting on the first rendered frame; the panel entrance's 8px `@starting-style` translateY is subtracted from the target so it lands on the resting row; ends with a fade, no row pulse), and is widened via `#summary #fCount > div` (`width: max-content`, 400px cap; must out-specify the shared 280px rule). The settings Filter row stacks in two lines (label, then the column select and value input sharing the next line; `#settingsPanel #fltRow` must out-specify `.row`'s `align-items: center`, and its `.ctl` is nowrap with a shrinking select)
- Popup pin: the geometry that opened a popup stays hover-highlighted until it closes (module-level `hl()`; `p.grp`/`cm.grp` tag permit-record groups so the whole group pins in permit view; `popupopen`/`popupclose` apply/release, `mouseout` guards skip the restore while pinned)
- No long dashes (en/em) in user-visible strings; use words ("to", ":", "n/a")
- Column display conventions: `%` suffix for names matching /pct|percent|rate/ plus `RATE_COLS` (derived conversion rates named without "rate"), `$` prefix for /amount|revenue/ (never /sale/); `DISPLAY` renames columns at the UI layer only (HomeCount shows as Homes, BadCount as Bad Leads) — internal CSV names feed every derivation, never rename those
- Metric dropdown and popup share one classifier (`classifyCol`); keep both grouped identically
- CityList and ZipOrFSA are excluded from popups; CityList values can be enormous and dirty (clamp behind a toggle); `HIDDEN_DISPLAY` (currently NoCount, AvgRevenuePer100Homes) is parsed but kept out of dropdown and popups; `HIDDEN_DROPDOWN` (currently AreaKm2) stays in popups but not the metric dropdown
- Units setting: Settings > Units switches area/density metrics between km² and sq mi (`UNITS` in dkm-prefs as `units`); first upload hints it (majority US zips = sq mi, else km²), the choice persists, the current metric swaps to its counterpart family, and the inactive family is hidden from dropdown, filter, and popups (`unitHidden`; group label follows via `densGroup()`); pinning a unit metric pins its other-unit twin too (`unitSwap`, unpins together, only when the twin exists)
- Derived metrics live in two places: CSV-derived (per-lead, per-contact, RevenuePer100Knocks) in `loadRows`, geometry-derived (AreaKm2, AreaSqMile, HomesPerKm2, HomesPerSqMile, per-km² and per-sq-mi outcomes) in `deriveDensity` which reruns when `boundaries.js` finishes lazy-loading; all density metrics (both unit families) group under the density group in the dropdown and popups (one classifier, `classifyCol`)

## Verification

- `node --check` on the extracted inline `<script>` (extraction one-liner lives in shell history of recent work; re-extract with python regex) before browser testing
- Serve with `python3 -m http.server 8901` and exercise in a browser (CDP via the `browser-use` harness; upload `example.csv` with `DOM.setFileInputFiles`, assert via `Runtime.evaluate` returning primitives only — returning DOM/Leaflet objects throws "Object reference chain is too long")

## Child DOX Index

- `data/AGENTS.md` — raw downloaded geo/census sources and extracted shapefiles (gitignored build inputs)
- Root-owned files: `index.html`, `build_centroids.py`, `build_boundaries.py`, `build_census_homes.py`, generated `centroids.js`/`boundaries.js`/`census_homes.js`, `README.txt`, `example.csv`, `sample_report.csv`, `demo_chicago_From_Apr_1_2026_To_Sep_20_2026.csv` (generated demo report, 47 real Chicago-area ZCTAs; season badge parses from the filename), the three WEMMS screenshots (`door-knocking.png`, `zip-fsa-report.png`, `report-settings.png`), `.gitignore`, `.env` (key only, never content)
