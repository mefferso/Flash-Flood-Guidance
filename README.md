# Flash Flood Guidance Map

GitHub-ready conversion of the LIX Flash Flood Event Mapper.

This version moves the project away from Google Apps Script callbacks and into a static GitHub Pages map powered by generated GeoJSON files.

## What this repo does

- Builds a flash flood event GeoJSON from the existing CSV or from NCEI Storm Events bulk CSV files.
- Hosts a Leaflet map on GitHub Pages.
- Adds optional critical infrastructure layers from public HIFLD/ArcGIS REST services.
- Includes a starter MRMS enrichment workflow that samples archived MRMS hourly QPE around each flash flood event.
- Leaves room for the eventual goal: a local, event-calibrated flash flood guidance / susceptibility layer.

## Suggested repo layout

```text
.
├── data/
│   └── raw/
│       └── lix_flash_flood_events.csv
├── docs/
│   ├── index.html
│   └── data/
│       ├── flash_flood_events.geojson
│       ├── flash_flood_events.csv
│       ├── critical_infrastructure.geojson
│       └── mrms_event_metrics.csv          # optional/generated later
├── scripts/
│   ├── build_events.py
│   ├── fetch_critical_infrastructure.py
│   ├── enrich_mrms.py
│   └── config.py
├── requirements.txt
└── .github/workflows/build.yml
```

## First local build

```bash
python -m pip install -r requirements.txt
python scripts/build_events.py --input data/raw/lix_flash_flood_events.csv
python scripts/fetch_critical_infrastructure.py --out docs/data/critical_infrastructure.geojson
```

Then open:

```text
docs/index.html
```

## GitHub Pages

1. Create a new GitHub repo.
2. Upload this repo.
3. In GitHub, go to **Settings → Pages**.
4. Set source to **GitHub Actions**.
5. Run the **Build and publish map** workflow.

## MRMS enrichment

MRMS is intentionally separated from the normal build because it can be slow as hell once you start sampling hundreds of events.

Start small:

```bash
python scripts/enrich_mrms.py \
  --events docs/data/flash_flood_events.geojson \
  --out docs/data/mrms_event_metrics.csv \
  --limit 25
```

Once that works, remove `--limit`.

## Important MRMS notes

- Use MRMS **hourly QPE** first, not instantaneous precip rate. Flash flooding is usually about short-duration accumulation and efficiency, not one radar-scan rate spike.
- For recent years, the Iowa State MTArchive product path usually uses `MultiSensor_QPE_01H_Pass2`.
- For older years, the older `GaugeCorr_QPE_01H` path may be available.
- The script tries both product names.
- Treat results as analysis guidance, not truth carved into stone tablets by Zeus. MRMS has quality issues, beam/blockage issues, Z-R issues, bright band issues, tropical vs convective issues, etc.

## End-state idea

The eventual guidance layer should probably be a gridded/hex layer with fields like:

- event_count
- severe_event_count
- critical_infra_count
- max_1h_event_qpe_p50 / p75 / p90
- max_3h_event_qpe_p50 / p75 / p90
- max_6h_event_qpe_p50 / p75 / p90
- normalized_event_rate
- recent_trend
- confidence_score

That gets you toward a locally calibrated “these places have flooded when MRMS looked like this” product instead of generic FFG that treats the world like a spreadsheet with a rain gauge taped to it.
