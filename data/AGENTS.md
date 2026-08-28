# data/

## Purpose

- Raw downloaded geo/census sources and extracted shapefiles used as build inputs for `centroids.js`, `boundaries.js`, and `census_homes.js`
- Nothing here ships with the app; everything is regenerable

## Ownership

- Owned by the root build scripts (`build_centroids.py`, `build_boundaries.py`, `build_census_homes.py`), which document their exact download commands in their headers
- Entire directory is gitignored

## Local Contracts

- GeoNames dumps: `US.txt`, `CA.txt` (inside `US.zip`, `CA.zip`), tab-separated, fields 2/10/11 = code/lat/lng
- Census ZCTA 2020: `zcta/` (extracted from `zcta.zip`, cb_2020_us_zcta520_500k)
- StatCan FSA 2021: `fsa/` (from `fsa.zip`, lfsa000b21a_e, Lambert projection, needs `-proj wgs84` in mapshaper)
- StatCan census profile: `98-401-X2021013_English_CSV_data.csv` (~620MB, latin-1 encoded, CHARACTERISTIC_ID 4 = total dwellings, 5 = occupied)
- `readme.txt` is StatCan's, not ours

## Work Guidance

- After refreshing any source, re-run the matching build script from the repo root; do not hand-edit generated JS files
- Keep intermediates (mapshaper output) in `/tmp/opencode/`, not here

## Verification

- Build scripts print feature counts; recent healthy runs: centroids 43,141 codes, boundaries 35,433 codes (~11.4MB), census homes 35,418 codes (~440KB)

## Child DOX Index

- None; no subdirectories with their own identity
