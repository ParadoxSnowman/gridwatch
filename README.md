# GRIDWATCH — FE Ohio outage / weather / data-center correlator

## What it does
Polls FirstEnergy Ohio's KUBRA StormCenter outage feed, enriches every outage
with trailing-3h weather (Open-Meteo) and active NWS alerts, computes distance
to each site in `datacenters.json`, classifies, logs to SQLite, and renders a
folium map. The `report` command answers the actual question longitudinally:
is the fair-weather outage rate inside DC proximity rings elevated vs the rest
of the territory?

## Classes
- **WEATHER-LIKELY** (blue) — gusts/wind/precip/temp-extreme/NWS alert/utility weather cause
- **DC-PROXIMATE FAIR-WEATHER** (red) — no weather signal, within ring; darker red if the utility's own cause string looks like dig-in/vehicle/construction damage
- **AMBIGUOUS** (purple) — both
- **UNEXPLAINED FAIR-WEATHER** (amber) — neither (equipment failure baseline)

## Setup
```
pip install requests folium
python gridwatch.py discover      # gets KUBRA GUIDs (DevTools fallback in --help)
# edit datacenters.json — verify coords against county auditor GIS
python gridwatch.py poll          # schedule every 15 min (Task Scheduler)
python gridwatch.py map           # writes gridwatch_map.html
python gridwatch.py report --days 30
```

## Windows Task Scheduler one-liner (adjust paths)
```
schtasks /create /tn GRIDWATCH /sc minute /mo 15 /tr "python C:\path\to\gridwatch.py poll"
```

## Analytical honesty checklist
1. Operating DCs are transmission-interconnected with flat load — they don't
   plausibly cause residential feeder outages directly. The testable mechanism
   is **construction phase**: dig-ins, pole strikes, substation cutovers,
   planned switching. Watch `status: construction` sites hardest.
2. One snapshot is noise. The signal is a persistent (weeks) elevated
   fair-weather rate in the rings, or a step-change when a site breaks ground.
3. Confounders: DC rings sit in exurban growth corridors — more construction
   of ALL kinds, different tree cover, older/newer feeders. If you see signal,
   the next move is a control ring (same county, no DC) before you claim
   anything.
4. Utility cause strings are self-reported; log, don't trust.
5. If the ratio holds up: PUCO reliability dockets (CAIDI/SAIFI by district
   are filed annually), and FE's large-load interconnection filings will name
   the substations being modified — that's your FOIA-adjacent paper trail.

## Known fragilities
- KUBRA GUIDs rotate on redeploys → rerun `discover`.
- currentState schema shifts occasionally; `_extract_data_path` searches the
  JSON tree rather than hardcoding, and dumps the payload if it fails.
- Columbus data center alley is **AEP territory** — not on this map. AEP also
  runs KUBRA; same scraper works with different GUIDs if you want to extend.


## GitHub Pages deployment (public map, no server)

Repo layout this package gives you:
```
gridwatch/
  gridwatch.py
  datacenters.json
  gridwatch_config.json      <- contains your KUBRA GUIDs (committed; they're public anyway)
  .gitignore                 <- keeps the local SQLite out of the repo
  .github/workflows/gridwatch.yml
  docs/
    index.html               <- static Leaflet map
    data/                    <- written by every poll (latest.json + history/)
```

Setup:
1. Create the repo, push everything.
2. Repo Settings > Pages > Source: "Deploy from a branch" > branch `main`, folder `/docs`.
3. Settings > Actions > General > Workflow permissions: "Read and write permissions".
4. Actions tab > gridwatch-poll > "Run workflow" once manually to verify, then the
   15-min cron takes over (GitHub delays scheduled runs a few minutes under load).
5. Map is live at https://<user>.github.io/<repo>/

Notes:
- CI mode uses `--no-db`: history accumulates as one NDJSON file per day in
  docs/data/history/ (git-friendly, analyzable with pandas later). Your local
  scheduled task keeps the SQLite for `report`.
- Multi-state: fill in instance_id/view_id for PA/NJ/MD/WV in
  gridwatch_config.json — DevTools on each state's outage page, exactly like OH.
  Regions with blank GUIDs are skipped with a notice.
- GitHub Actions scheduled workflows get disabled after 60 days of repo
  inactivity; the bot commits count as activity, so this self-sustains.

## Framing note for the public page

The outage map measures *reliability*. The "data centers straining the grid"
story most people mean is *rates and capacity* (PJM capacity auction costs
socialized onto ratepayers, FE's transmission capex passed through bills) —
that does not show up as outage dots. Keep the two claims separate on the
public site or the first utility PR person to look at it will use the
conflation to dismiss the whole thing. The outage layer tests one narrow,
falsifiable claim: construction-phase distribution stress near sites.


## v2 additions
- **History tab**: per-day snapshot browser, deduplicated across polls, hourly sparkline.
- **Evidence tab**: fair-weather share in DC rings vs elsewhere over the last 30 days,
  ratio with honesty-gated verdict, per-site scorecard (CONTROL sites participate).
- **Auto-OSINT layer**: `python gridwatch.py osint` (and automatically, weekly, during
  CI polls) queries OpenStreetMap Overpass for mapped data centers across all region
  bboxes -> docs/data/datacenters_auto.json. Context layer only (OSM lags new
  construction); toggle it on the map from the site list. Your curated
  datacenters.json remains the evidence layer.
- Multi-state: fill instance_id/view_id for PA-NY / NJ / MD / WV in
  gridwatch_config.json via DevTools on each state's outage page. Note the PA map
  covers PA+NY; MD and WV may share a deployment - if outages-md and outages-wv
  redirect to the same view GUID, keep one region entry with a merged bbox.
