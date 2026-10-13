# signalk-livedepth-tiles

The shared gridded-bathymetry tile algorithm behind the **LiveDepth** feature.
One implementation, used by both sides so they build and read identical tiles:

- the server plugin [`signalk-livedepth-plugin`](https://www.npmjs.com/package/signalk-livedepth-plugin), and
- the chart client (Freeboard-SK's LiveDepth layer).

## What's in it

- **`grid`** — Web-Mercator XYZ quadtree geometry and the `sparse-v1` wire codec.
  256-cell tiles; base level `z20` (≈0.15 m/cell), min-pooled down to `z4`
  (≈2.4 km/cell). A tile is a sparse list of occupied cells — 5 bytes each
  (`cx:u8, cy:u8, depthCm:u16le, count:u8`) — keeping the **shallowest** depth,
  so zooming out never deepens a shoal. The codec uses `DataView`/`Uint8Array`,
  so it runs under Node and in the browser.
- **`store`** — the `TileStore` interface and an in-memory `Map` implementation.
  The plugin supplies a SQLite-backed store; the client the in-memory one.
- **`accumulator`** — `createTileAccumulator` folds soundings into the base
  tiles and min-pools their ancestors (with a deferred mode + `rebuildAncestors`
  for bulk loads).
- **`tile-source`** — read-side helpers: pick the tile level for a target ground
  resolution, work out which tiles cover a viewport, and decode a served tile
  back into lon/lat soundings.

Pure TypeScript, no runtime dependencies, no Node/browser-specific APIs.

## Use

```ts
import {
  createTileAccumulator,
  createMemoryTileStore,
  tileRangeForBbox,
  decodeTileSamples,
} from "signalk-livedepth-tiles";
```

## Development

```bash
npm install
npm run build
npm test
npm run format
npm run lint
```
