# housing-economic-shards

Per-geography JSON shards for **[housing-economic-data](https://github.com/jayschellenberg/housing-economic-data)**.
This repo holds data only — no code, no build.

| Directory | Contents | Emitted by |
|---|---|---|
| `data/series/` | CMHC Rms long-form shards, one per geography (`{level}_{uid}.json`) | `r/03_build_data_files.R` |
| `data/starts/` | CMHC Scss housing-starts shards, one per geography | `r/03_build_data_files.R` |

## Why these live outside the app repo

Vercel stores the full build output of **every retained deployment** separately —
there is no de-duplication between deployments (measured 2026-09-07: 36.14 GB
across the team, against a 10 GB tier). These shards are ~401 MB. Committed to
the app repo they were copied into `dist/` on every build, so each deployment
cost ~425 MB even though a typical commit changes only one or two shards.

Serving them from here instead takes the app's build output to ~25 MB. The app
fetches them through its own edge-cached proxy at `/gh-data/...`, which pins an
immutable commit SHA — so a shard URL never changes content, and publishing a
refresh is a matter of pushing here and bumping one constant in the app.

## Publishing a refresh

1. Run the app repo's data pipeline (`npm run data:shards` in `web/`).
2. Copy the regenerated `series/` and `starts/` trees here, commit, push.
3. Bump `SHARDS_REVISION` in `web/src/shards.js` in the app repo to the new
   commit SHA and push. Until that bump, production keeps serving the old pin —
   which is the point: the app never sees a half-published tree.
