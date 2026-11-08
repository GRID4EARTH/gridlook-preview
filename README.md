# Gridlook preview: upstream main + the multiscales draft + GRID4EARTH fixes

**Temporary preview, not a release.** A static production build of Gridlook that puts together pieces that are
not deployed together anywhere else yet:

- upstream [d70-t/gridlook](https://github.com/d70-t/gridlook) `main` at `64eb9b0` (2026-10-30);
- the draft pull request `d70-t/gridlook#235` (multiscales levels, zoom-driven level selection and view-based loading,
  by espg) at its head `0e7c178` (branch `upstream/levels-lod` of espg/gridlook, 2026-10-23), with that `main` merged in;
- our fixes, each on its own branch of tinaok/gridlook (table below), opened or to be opened upstream.

Rebuilt on 2026-11-08 with `npm run build`. Source maps are left out. Gridlook is MIT-licensed (see the upstream
repository).

It is published so that GRID4EARTH can try its multi-resolution HEALPix products, including the JPEG XL compressed
Sentinel-2 L2A reflectance, before these changes are merged upstream.
Discussion: [GRID4EARTH/project-guidelines#21](https://github.com/GRID4EARTH/project-guidelines/issues/21).

## Open

| What | Link |
|---|---|
| 76-cell Sentinel-2 L2A orbit mosaic (Icechunk), b02 at level 20: the colour range is widened past the coarsest level's | <https://grid4earth.eu/gridlook-preview/#icechunk+https://data.grid4earth.eu/test-orbit-mosaic/S2B_MSIL2A_20260812T104619_R051_O49269_fullsphere.icechunk/measurements/reflectance::varname=20/b02::alt=2500::lat=46.8::lon=0.8> |
| Copernicus DEM GLO-30 over Europe, levels 18 (25 m) to 8, compacted cell ids, Mont Blanc | <https://grid4earth.eu/gridlook-preview/#https://data.grid4earth.eu/auxiliary/copernicus-dem-glo-30/copernicus-glo-30-v0-europe-pyramid.zarr/dem::boundlow=500::boundhigh=4800::lat=45.8326::lon=6.8652::alt=60000> |
| Sentinel-2 L2A cell, JPEG XL (hcc) codec, b05 (native at level 19: level 20 is greyed out) | <https://grid4earth.eu/gridlook-preview/#https://data.grid4earth.eu/converted-healpix-tiled/sentinel-2-l2a-multires/mean-l20-l12-hcc/S2B_MSIL2A_20260812T104619_R051_O49269_L7_C10975.zarr/measurements/reflectance::varname=19/b05::alt=3000::lat=46.31::lon=3.46> |
| Copernicus DEM GLO-30 Europe mosaic (Icechunk), level 12 over the North Sea: open sea without DEM tiles shows as no data, not as the coarsest level | <https://grid4earth.eu/gridlook-preview/#icechunk+https://data.grid4earth.eu/converted-mosaic/copernicus-dem-glo-30/mean-l18-l8-c8-s6/l5/copernicus-dem-glo-30-europe.icechunk/dem::varname=12/dsm::px=0::py=0::alt=533672::lat=56.3475::lon=1.73::layers=mask,1,0.6;coastlines;grid::boundlow=0::boundhigh=1348> |
| Kerchunk reference file with levels (2-cell mosaic) | <https://grid4earth.eu/gridlook-preview/#reference+https://data.grid4earth.eu/test-orbit-mosaic/kerchunk/S2B_MSIL2A_20260812T104619_R051_O49269_fullsphere_C10974-C10975.json/measurements/reflectance> |
| RGB composite layer, Plouzané 2018-09-17 | <https://grid4earth.eu/gridlook-preview/#https://data.grid4earth.eu/test-orbit-mosaic/plouzane_timeseries/rgb_level20_dn_zstd.zarr::varname=rgb::dimIndices_time=155::layers=rgb,1,1,band,0,1,2,0,0.3;coastlines;grid> |
| RGB composite on a full-sphere pyramid (levels 20 to 12, read by view), Plouzané 2018-09-17: zoom in | <https://grid4earth.eu/gridlook-preview/#https://data.grid4earth.eu/test-orbit-mosaic/plouzane_timeseries/rgb_fullsphere.icechunk::dimIndices_time=155::layers=rgb,1,1,band,0,1,2,0,0.3;coastlines;grid::lon=-4.615::lat=48.385::alt=20000> |
| RGB composite from three variables (b04, b03, b02) of a Sentinel-2 L2A cell, hcc codec | <https://grid4earth.eu/gridlook-preview/#https://data.grid4earth.eu/test-integ-20261030/converted-healpix-tiled/sentinel-2-l2a-multires/mean-l20-l12-hcc/S2C_MSIL2A_20260813T110621_R137_O10117/S2C_MSIL2A_20260813T110621_R137_O10117_L7_C54537.zarr/measurements/reflectance::varname=b04::layers=rgb,1,1,@vars,b04,b03,b02,0,0.3;coastlines;grid> |

Since `0e7c178` the level is part of the variable name in links (`varname=19/b05`). A link from before, with a bare
name that the first (finest) level does not hold (`varname=b05` on this cell), does not open; give the level.

## What was built

On upstream `main` `64eb9b0`, merged into `0e7c178` (three conflicts: `sourceIndexing.ts`, `ZarrDataManager.ts`,
`GaussianReduced.vue`):

| commit | what | upstream |
|---|---|---|
| `fbc9e4d` | the shared colour range of the levels ("Keep one colour range across levels", on) starts from the coarsest level and only widens to the levels shown, so fine levels of means do not saturate | `espg/gridlook#29` (branch `fix/multires-colour-range`) |
| `68dabda` `e19c399` | multiscales layouts that list arrays (`"asset": "18/b02"`, written by healpix-convert since GRID4EARTH/healpix-convert#71) are read as one level per parent group, also when the store is opened from its root | `espg/gridlook#30` (branch `feat/array-asset-layout-on-235`) |
| `de6e5e2` `7f92e86` | HEALPix levels whose cell ids use the dggs `compression` `compacted` (zuniq) or `ranges`, loaded by view without one stored id per cell | `espg/gridlook#31` (branch `feat/dggs-compacted-cells-on-235`) |
| `ab0981b` | dense HEALPix faces are meshed over their finite values only (they were drawn up to about 230 m off) | branch `fix/dense-healpix-data-rect`, for `main` |
| `bbdfbc1` | a Zarr v2 `fill_value` of 0 marks missing data, as in xarray (Zarr v3 unchanged) | branch `fix/fill-value-zero`, for `main` |
| `71832d9` | initial colour range from CF `actual_range`, else `valid_range` / `valid_min` + `valid_max` | branch `feat/default-range-from-attrs`, for `main` |
| `976ea6e` `231a2d9` `4be622a` | RGB composite layer (three entries of one dimension as a true-colour image); only band-like dimensions are offered (not time, 3 to 32 entries) | `d70-t/gridlook#252` (draft) |
| `c529bcf` `b3d6fc6` | RGB composite from three variables of the current variable's group (Source: Three variables, e.g. b04/b03/b02 without a quicklook); on a pyramid, the variables of the level shown | branch `feat/rgb-three-variables-preview` of tinaok/gridlook (`feat/rgb-three-variables` for `main`), not proposed upstream (see `d70-t/gridlook#255`) |
| `2a1f5f4` | the RGB layer reads only the blocks in view on large levels (it read whole faces and failed), and reloads with the level | same branch |
| `d482d3a` `1ef5c9a` `896d69b` `4302ddd` | decoders of the healpix-compress Sentinel-2 codecs, now from the `zarrita-hcc` package at `v0.1.2` (JPEG XL loaded only when a dataset uses them; the in-tree copy from GRID4EARTH/gridlook#8 is reverted) | branch `feat/hcc-codecs-package`, for `main` |
| `4842755` | quality selector for the automatic multiscales level (Auto / Fast / Standard / Sharp; `pxpercell` in the link overrides it); a coarser level is applied at once, a finer one after the camera rests 350 ms (was GRID4EARTH/gridlook-preview#13) | experiment, not proposed upstream; whole source of this build: branch `preview-source-2026-11-08` of tinaok/gridlook |
| `2a499b0` | the coarsest level drawn under a multiscales level (the backdrop) stays out of where that level has cells, so missing values of the level show as no data instead of the coarse level's colour; the backdrop still fills blocks not loaded yet | branch `fix/backdrop-cover-mask` of tinaok/gridlook, on top of `4842755`; not proposed upstream yet |
| `9433e74` `883e380` | find an xdggs `cell_ids` coordinate when no dggs coordinate is named (port of `af1ff86` onto `0e7c178`) | branch `fix/healpix-cell-ids`, for `main` |
| `0098f63` `32592ed` | GRID4EARTH only: ERA5 2 m temperature by default, dataset list from `https://data.grid4earth.eu/gridlook-catalog.json` | — |

After the merge of `main`, 9 tests of `0e7c178` fail only because their mocks predate `main` (no page URL for the
Jupyter `_xsrf` token in `GlobeView.test.ts`, no `HttpStorage` in one Icechunk test); everything else passes, with
`vue-tsc`, `eslint --max-warnings 0` and `prettier`.

## Dataset list and default dataset

Same as grid4earth.eu/gridlook (GRID4EARTH/gridlook#2, #6). The "Open dataset" list is read from
`https://data.grid4earth.eu/gridlook-catalog.json`, which `stac-scraper gridlook-catalog` writes with the STAC
metadata (GRID4EARTH/stac-scraper#28). Opened without a dataset in the URL, this build shows ERA5 2 m temperature.

## Update

Rebuild from the commits above and replace the files:

```bash
npm ci && npm run build
rsync -a --delete --exclude '*.map' --exclude .git --exclude README.md --exclude .nojekyll dist/ <this repository>/
```

Once these changes are merged and deployed at <https://grid4earth.eu/gridlook/>, delete this repository or reuse it
for the next preview.
