GRIDWATCH

A public-interest instrument for holding electricity monopolies accountable.
## Are data centers sucking the life outta our grid? 
<img width="523" height="382" alt="image" src="https://github.com/user-attachments/assets/b8aa2167-6b58-468e-a710-cc9595e4ebb0" />

Live: https://paradoxsnowman.github.io/gridwatch/ · Methodology: about.html

Most Americans cannot choose who delivers their electricity. A single regulated utility owns the wire, decides what to tell you when the power goes out, and reports its own reliability once a year using a method that lets it exclude its worst days.

GRIDWATCH polls 34 utility outage maps from Michigan and Maine down to Georgia, every few minutes, and keeps what the utilities themselves publish — then makes it comparable. It asks three questions:

How reliable is your monopoly, really? Continuous measurement instead of an annual self-report.
How much do they actually tell you? A disclosure grade per utility, and whether the restoration estimates they publish turn out to be true.
Is data center construction shifting cost and reliability onto ordinary customers? Outage proximity to active construction, tested against matched control corridors, with the capacity-market receipts alongside.
What it does

Type in your ZIP code and get what GRIDWATCH has recorded within 15 km over the trailing 30 days: incident count, fair-weather share, the most common reported cause, your area's worst repeat-failure locations with coordinates, a percentile rank against every other area tracked, who your utility is, which commission regulates them — and a CSV export you can attach to a complaint.

Live map — every outage colour-coded by weather-likely, fair-weather, near active data-center construction, near an operating facility (context only), or planned utility work. Animated NEXRAD radar overlays it so you can audit the weather classification yourself. Historical playback scrubs day by day with restoration times.

Accountability tab — the disclosure scorecard (A–F per utility), ETR accuracy, customer-minutes interrupted, chronic repeat-failure locations, and GRIDWATCH's audit of its own feeds.

Evidence tab — active-vs-control comparison with 95% Wilson confidence intervals that refuses to claim a difference until the data supports one.

Burden tab — PJM capacity-auction clears, zonal demand growth, the interconnection queue near tracked sites, retired plants whose interconnection rights a campus can inherit, and dated regulatory decisions worth watching.

Honesty rules, enforced in code
Planned work is excluded from the evidence. Utilities schedule maintenance for good weather, so it lands in the fair-weather bucket by design — measured at ~16% of all fair-weather outages. It is tagged, shown on its own map layer, and removed from every evidence calculation.
Controls are published alongside. Matched growth corridors with no data center. If they run as hot as real sites, the honest answer is "construction corridor", and the page says so.
Null results publish as prominently as positive ones.
The instrument audits itself. Every poll checks each feed for a frozen endpoint, geographic drift, missing customer counts, silence, and volume far below its own baseline. Failing feeds are flagged suspect on the site and excluded from the statistics rather than quietly averaged in. This caught AEP Ohio serving a year-old cached snapshot, and a layer-selection bug that had blanked northern West Virginia.
Restricted data stays out. True grid topology is CEII; the lines and substations shown are OpenStreetMap's public reconstruction, labelled as such. PJM's licensed data is not redistributed.
Data sources
layer	source	terms
Outages	34 utility outage maps	© respective utilities
Weather	Open-Meteo + NWS alerts	public / public domain
Reliability (SAIDI/SAIFI)	EIA-861	US public domain
Demand & forecast	EIA-930	US public domain
Retirements	EIA-860M	US public domain
Interconnection queue	PJM planning queue	public
Zone prices	NYISO public CSVs	public
Grid topology	OpenStreetMap	ODbL
Basemap	Esri (OSM fallback)	keyless

Five feed protocols are handled: KUBRA StormCenter, legacy iFactor, Duke's Apigee REST API, PPL's OMAP, Avangrid's APIM, ArcGIS MapServer, and two single-file XML feeds.

Open data

Everything the poller produces is downloadable and reusable:

docs/data/latest.json — current snapshot, classified
docs/data/latest.geojson — same, for QGIS/ArcGIS/Felt
docs/data/feed.xml — RSS digest, per poll and weekly
docs/data/hotspots.json — 30-day spatial rollup
docs/data/accountability.json — disclosure grades, ETR accuracy, CMI
docs/data/feed_audit.json — our own data-quality report
docs/data/daily/ — per-day incidents with restoration times
docs/data/history/ — raw NDJSON, one file per day
Running it yourself
bash
pip install requests
python gridwatch.py verify            # test every configured feed
python gridwatch.py poll --emit docs/data --no-db

Optional repo secrets (everything degrades gracefully without them):

secret	unlocks
DUKE_BASIC_TOKEN	Duke's four jurisdictions
EIA_API_KEY	demand, unforecast load, retirements, SAIDI scorecard
AVANGRID_KEY	NYSEG, RG&E, Central Maine Power
PJM_API_KEY	PJM zonal prices (private use — DM2 forbids redistribution)

GRIDWATCH_REGIONS=OH,PA-NY scopes a run to specific feeds. A region can be parked with "enabled": false without losing its config.

Never hand-edit docs/data/ — the outage history lives there.

Adding a utility

Most outage maps are one of a handful of platforms. Open the utility's map, DevTools → Network, filter by the utility's own domain (analytics and Google Maps drown everything else), and look for:

currentState → KUBRA. Paste the URL; the instance and view GUIDs are in it.
/query? on an ArcGIS service → use the arcgis provider with its base URL.
a single .xml or .json from the utility's host → the generic XML provider.

Add a region to gridwatch_config.json with a bounding box and it polls on the next run.

Caveats

This is one person's public-interest project, not an official source. Counts are GRIDWATCH's observations of public outage maps, not utility filings. Customer figures come from what utilities publish and several publish bucketed ranges rather than exact counts. Florida and other hurricane-dominated territories are deliberately excluded — storm noise would swamp the fair-weather signal, and scope honesty is worth more than coverage.

Corrections and scrutiny are welcome. Every number links back to its origin so anyone can check it.


