# Gridlook preview: upstream main + d70-t/gridlook#235 + GRID4EARTH/gridlook#8

**Temporary preview, not a release.** This is a static production build of Gridlook with three pieces that are
not deployed together anywhere else yet:

- upstream [d70-t/gridlook](https://github.com/d70-t/gridlook) `main` at `6370a99` (includes #231, the info-panel rework, and #240, the basemaps);
- the draft pull request [d70-t/gridlook#235](https://github.com/d70-t/gridlook/pull/235) ("zoom-driven level
  selection and view-based loading for multiscales datasets", by espg) at `f45b093`, merged into that `main`;
- the decoders of [GRID4EARTH/gridlook#8](https://github.com/GRID4EARTH/gridlook/pull/8) (by jmdelouis) for the
  healpix-compress Sentinel-2 codecs `hcc-sentinel2-pyramid` and `hcc-sentinel2` (JPEG XL through `jxl-oxide-wasm`,
  loaded only when a dataset uses them), ported from the old `healpix-geo` base onto today's code.

Rebuilt on 2026-10-16 (after #240) with `npm run build` (node 24). Source maps are left out. Gridlook is MIT-licensed (see the
upstream repository).

It is published so that GRID4EARTH can try multi-resolution HEALPix products, including the JPEG XL compressed
Sentinel-2 L2A reflectance, before #235 and the codecs are merged upstream.
Discussion: [GRID4EARTH/project-guidelines#21](https://github.com/GRID4EARTH/project-guidelines/issues/21).

The earlier preview of #235 alone, [grid4earth.eu/gridlook-pr235](https://grid4earth.eu/gridlook-pr235/)
([GRID4EARTH/gridlook-pr235](https://github.com/GRID4EARTH/gridlook-pr235)), stays as it is: the STAC items of
`converted-sentinel-2-l1c-multires` still open there.

## Open

<https://grid4earth.eu/gridlook-preview/#https://data.grid4earth.eu/converted/sentinel-2-l1c-multires/mean-l20-l12/S2C_MSIL1C_20250514T104641_N0511_R051_T31UDQ_20250514T143130.zarr/measurements/reflectance>

Zoom in on Paris: with "Pick the level from the zoom" on (Variable card), the level switches from 13 to 20.
For a map underneath, use "Add layer…" → "Maps and Masks" (OpenStreetMap, Simple, Satellite) and move it below the data grid.

A Kerchunk reference file (2-cell orbit mosaic, full-sphere layout; GRID4EARTH/orbit-mosaic `docs/alternatives.md`):

<https://grid4earth.eu/gridlook-preview/#reference+https://data.grid4earth.eu/test-orbit-mosaic/kerchunk/S2B_MSIL2A_20260812T104619_R051_O49269_fullsphere_C10974-C10975.json/measurements/reflectance::boundlow=0::boundhigh=0.3>

Copernicus DEM GLO-30 over Europe, levels 18 (25 m) to 8, compacted cell ids; the colour range starts from level 8,
so give it in the URL:

<https://grid4earth.eu/gridlook-preview/#https://data.grid4earth.eu/auxiliary/copernicus-dem-glo-30/copernicus-glo-30-v0-europe-pyramid.zarr/dem::boundlow=500::boundhigh=4800::lat=45.8326::lon=6.8652::alt=60000>

No JPEG XL compressed Sentinel-2 L2A store is public yet; once one is under
`https://data.grid4earth.eu/converted/sentinel-2-l2a/`, it opens the same way.

## What was built

The source is not published as a branch. On top of upstream `main` at `6370a99`:

| commit | what |
|---|---|
| `5304f69` | merge of #235 (`f45b093`) into `main`; 9 conflicted files (mostly #240 basemaps vs #235 view-window loading), and the info panel now follows the selected level |
| `9ce4099` `5d5f87e` `3f70e8b` | PR #8, ported: the codecs (registered in `src/lib/data/codecs.ts`, so also in the data workers), an end-to-end test through zarrita, `docs/codecs.md` (author jmdelouis) |
| `b6c7069` `01b085f` | tests: a DGGS store without `healpix_nside`, and a 2-level DGGS pyramid |
| `22bc0c1` `5ae2494` | GRID4EARTH only: ERA5 2 m temperature by default, dataset list from `https://data.grid4earth.eu/gridlook-catalog.json` |
| `f750fa9` | fix of `sortedCells.lowerBound`: `(lo + hi) >> 1` wrapped past 2**31 cells (whole-orbit mosaic, levels 19-20) |
| `dd6e18d` | Kerchunk reference files, opened as `reference+https://<file>.json[/<group>]` (branch `feat/kerchunk-reference` of tinaok/gridlook, on upstream `main`, for an upstream pull request) |
| `b795778` `1b09e9d` | merge of that branch, and the multiscales levels for reference files |
| `6de218d` | colour range of a multi-resolution dataset: the coarsest level's range widened to the level on screen (only widens per dataset, variable and slice), so fine levels no longer saturate; upstream as `espg/gridlook#29` (against #235's branch) |
| `aa817a1` `778b875` | HEALPix levels whose `cell_ids` use the dggs `compression` `compacted` (zuniq ids) or `ranges`: loaded by view without one stored id per cell (same as `f28c5f0` `21af6ff` of branch `feat/dggs-compacted-cells`; upstream draft against #235's branch) |
| `85deaa1` | initial colour range from the variable's CF attributes `actual_range`, else `valid_range` / `valid_min` + `valid_max` (unpacked with `scale_factor` / `add_offset`), when neither the URL nor a JSON index gives one; read where the range is used, so it works on every level (branch `feat/default-range-from-attrs` of tinaok/gridlook, `8cbb78d` on upstream `main`, for an upstream pull request) |

Differences from PR #8: its own `numcodecs.delta` decoder is not needed (zarrita 0.7.5 ships one), and neither is its
`healpix_nside` fallback (`main` already reads the DGGS `refinement_level`); `1b6999f` tests that instead.

The codec commits (`9ce4099` to `01b085f`) are meant to become an upstream pull request once #235 is merged.

## Dataset list and default dataset

Same as grid4earth.eu/gridlook (GRID4EARTH/gridlook#2, #6). The "Open dataset" list is read from
`https://data.grid4earth.eu/gridlook-catalog.json`, which `stac-scraper gridlook-catalog` writes with the STAC
metadata (GRID4EARTH/stac-scraper#28): one checked example per STAC collection except the mirrors. Opened without a
dataset in the URL, this build shows ERA5 2 m temperature (`auxiliary/reanalysis-era5-complete`).

## Update

Rebuild from the commits above and replace the files:

```bash
npm ci && npm run build
rsync -a --delete --exclude testdata --exclude '*.map' --exclude .git --exclude README.md --exclude screenshots --exclude .nojekyll dist/ <this repository>/
```

Once #235 and the codecs are merged and deployed at <https://grid4earth.eu/gridlook/>, delete this repository or
reuse it for the next preview.
