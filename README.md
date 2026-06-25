# Vina Subbasin — Where the Losses Happen (2027 BC RMS Network · 26 polygons)

A drought-conditioned look at groundwater storage in the Vina Subbasin,
prepared by **AGUBC** for the Vina GSA technical group and AGUBC Board.

The polygons, RMS wells, and DWR periodic GWL measurements all come from the
companion
[2027-BC-prop-network](https://agubc-vina.github.io/2027-BC-prop-network/)
framework. The dashboard reads the **three-zone (per–management area)
tessellation**: three independent Voronoi tessellations — one each for
Vina-North, Vina-Chico, and Vina-South — each clipped to its own
management-area boundary, so polygon cells never cross management-area lines.

> **Why three-zone only?** The three management areas carry distinct
> sustainable management criteria (especially for subsidence), so a polygon's
> hydrology must roll up to the zone where the well physically sits. An earlier
> build also offered a single basin-wide tessellation whose cells crossed
> management-area lines; that method has been retired.

## The question this dashboard answers

> **When and where is the Vina Subbasin losing groundwater storage, and how
> big is the deficit relative to what's in the basin?**

## Headline finding

Loss is concentrated in drought years, not spread evenly. Across WY 1999–2025:

- **Critical + Dry years remove about 466k AF**; **Wet + Above-Normal years
  recover about 290k AF.** The basin is already recharging in wet years — just
  not fast enough to offset the drought-year drawdown on its own.

Two basin totals are reported (see
[Year-type-weighted normalization](#year-type-weighted-normalization) for why):

- **Observed** basin net deficit through WY 2025: **−198,094 AF** — what the
  data show directly, with late-baseline polygons contributing only the years
  they observed.
- **Normalized** basin net deficit through WY 2025: **−234,838 AF** — what the
  basin would show if every polygon had a full WY 1999–2025 record, computed by
  year-type-weighted backcast from each polygon's own data.

Both are small relative to the **16+ MAF** of fresh groundwater in storage
(2022 Vina GSP, p. ES-5): observed is **1.24%** of total storage, normalized is
**1.47%**. The observed **WY 2022 trough** reaches **−342,214 AF** (≈2.14% of
total storage) — and even at that low, no SGMA sustainability indicator
registered an undesirable result.

| Condition                  | Years | Total ΔStorage (AF) | Avg per year |
|----------------------------|------:|--------------------:|-------------:|
| Wet                        | 6     | **+269,638**        | +44,940      |
| Above Normal               | 4     | **+20,348**         | +5,087       |
| Below Normal               | 5     | **−22,239**         | −4,448       |
| Dry                        | 6     | **−219,447**        | −36,574      |
| Critical                   | 5     | **−246,394**        | −49,279      |
| Net basin (WY 1999–2025)   | 26    | **−198,094**        | —            |

Year-type classification uses DWR's official **Sacramento Valley Index**
(Northern Sierra 8-Station Index).

## The polygon set

The 2027 BC RMS network is **26 polygons** in the three-zone tessellation:
**13 Vina-North, 1 Vina-Chico (aggregate), 12 Vina-South**. Polygons are
clipped to the DWR B118 Vina Subbasin boundary and projected through
EPSG:3310 (NAD-83 California Albers Equal Area). The roster comes from the
`2027-BC-prop-network` build, tied to the `2027 GWL RMS?` column of the
network workbook.

**Chico is a single aggregate polygon.** Rather than splitting Chico into
separate Voronoi cells, the Chico management area is one dissolved polygon
whose groundwater elevation is the composite of **10 nested well completions**
(`CWSCH01b` as the primary RMS site plus 9 supplemental completions, including
the `22N01E28J` cluster). Storage for Chico uses that composite elevation
against the dissolved polygon's full area.

**Spatial reassignment.** Each seed well is assigned to a management area by
spatial containment in the management-area boundary polygons, not by its
workbook tag. **Two wells** — `22N01E20K001M` and `22N01E09B001M` — are
workbook-tagged Vina-Chico but sit spatially in Vina-North, so the dashboard
shows them as North. The audit trail (`workbook_mgmt_area` + `reassigned` flag)
is preserved on every polygon for transparency, and reassigned polygons are
called out in the dashboard.

**Baseline anchoring.** Each polygon is baseline-anchored to the first
WY 1999–2025 year with a Good-quality March measurement at its RMS well.
**Eleven polygons anchor to WY 1999;** the other 15 baseline later (between
2000 and 2019) because their 2027 RMS well wasn't measured in 1999.

## Method, in brief

**Change in storage** for a polygon, between its baseline year and WY 2025, is:

```
change in storage = change in elevation × polygon area × Sy
ΔStorage_p = (GWE_p,end − GWE_p,baseline) × Area_p × Sy_p
```

This is exactly the calculation shown — with the actual values plugged in —
when you click a polygon on the map. For example, the Chico aggregate:

```
change in storage = −23.07 ft × 29,718 ac × 0.0958 = −65,681 AF (cumulative)
```

- **Specific yield (Sy) is polygon-by-polygon**, derived from DWR's SVSim
  Texture Data (Sacramento Valley Simulation Model v1.0, CKAN resource
  `544623e2-0cd5-4c5b-827f-affa4abf4e16`). Coarse-grained sediments → 0.15,
  fine-grained → 0.05, area-weighted by borehole lithology in the 0–500 ft
  below-ground-surface analysis window. **This is a measured, data-driven Sy —
  not an assigned constant like a flat 0.10.** Each polygon's value is built
  from the actual coarse/fine sediment fractions logged in the boreholes that
  fall inside it: **230 boreholes lie within the basin** (80 of them with
  ≥200 ft of logged lithology), and polygon Sy values range **0.0594 to
  0.1172** (basin area-weighted mean ≈ 0.087) — real spatial variation that a
  single assigned number would erase. The per-polygon popup shows exactly how
  many boreholes back each Sy. Three polygons with too few boreholes
  (`23N01E29P002M`, `22N01E09B001M`, `21N01E25K001M`) fall back to the basin
  area-weighted mean, flagged "(mean)" in the per-polygon detail table. See
  [`scripts/build_sy_svsim.py`](scripts/build_sy_svsim.py) for the full
  pipeline.
- **Groundwater elevation (GWE)** is the spring composite for each polygon's
  2027 GWL RMS well — March mean for SWN-named wells, Feb–Apr mean for CWSCH
  wells — using Good-quality DWR records only. Levels are reported in **feet
  above mean sea level (ft msl)** by default, so a *declining* water level is a
  *negative* change in elevation and a *loss* of storage. The popup can also
  show levels as **depth below ground surface (ft BGS)** — derived from each
  record's depth-to-water (ground-surface elevation = GWE + depth-to-water) —
  via the toolbar toggle; in that view a *deeper* water level (larger ft BGS)
  is the loss.
- **Gap attribution.** When a polygon has a multi-year gap in DWR Good
  measurements, the cumulative storage delta across the gap is distributed
  evenly across the missing years and bucketed by each year's hydrologic
  condition.
- **Hydrologic year classification** uses DWR's official Sacramento Valley
  Index (Northern Sierra 8-Station Index) — Wet / Above Normal / Below Normal /
  Dry / Critical.

## Year-type-weighted normalization

### The problem this corrects

Of the 26 polygons, only **11 have a Good-quality March measurement in WY
1999** and can baseline there. The other 15 baseline later — between 2000 and
2019 — because their 2027 GWL RMS well wasn't measured in 1999. Each polygon
contributes year-over-year deltas only for the years it has a baseline-anchored
record, so late-baseline polygons cannot register their pre-baseline drawdown.

| Baseline year | Polygons |
|---|---|
| WY 1999 | 11 |
| 2000 | 4 |
| 2002 | 4 |
| 2003 | 1 |
| 2008 | 1 |
| 2009 | 2 |
| 2011 | 1 |
| 2012 | 1 |
| 2019 | 1 |

This creates a real bias: the basin's **observed** 1999–2025 cumulative
*understates* the deficit, because polygons that started drawing down before
their first measurement get a "free pass" on their pre-baseline losses.

### The normalization method (year-type-weighted backcast)

For each polygon, using only its own observations, compute an average ΔStorage
rate *per Sacramento Valley Index year type*:

```
rate_p,t = (sum of polygon p's ΔStorage in years of type t)
           ÷ (# years of type t the polygon observed)
```

Then synthesize what each polygon would have contributed across the full record
by applying its per-type rates to the basin's actual year-type mix:

```
normalized_cum_p = Σ over t of (rate_p,t × N_t)
```

where `N_t` is the count of each year type in the WY 2000–2025 transition
window: **6 Wet, 4 Above Normal, 5 Below Normal, 6 Dry, 5 Critical = 26 years**.
Summing across all 26 polygons gives the normalized basin total.

**Fallback for unobserved year-types.** If a late-baseline polygon never
observed a given year type, its own overall average rate is substituted for
that year type, flagged "(fb)" in the detail table. In the current data this
affects one combination: `23N01E07H001M` (baselines WY 2019) has not observed a
Below-Normal year, so its overall average rate stands in for its BN rate. The
effect on the basin total is small.

### Why this is defensible

- **Each polygon uses only its own observations** — no proxying from
  neighboring wells, no model fill.
- **Captures the dominant hydrologic signal.** Drought-year loss rates and
  wet-year recovery rates are fundamentally different; a simple "AF/yr × years"
  multiplier would understate that asymmetry, while year-type weighting
  preserves it.
- **The math is fully transparent.** Every polygon's per-type rates are in
  `data/condition_analysis_three_zone.json`, and the basin total can be
  re-derived by hand.

### Headline impact

| | Observed | Normalized | Delta |
|---|---:|---:|---:|
| Basin cumulative WY 2025 | −198,094 AF | **−234,838 AF** | +36,744 AF more deficit |
| Basin avg loss rate | 8,701 AF/yr | **9,032 AF/yr** | +331 AF/yr more loss |

The normalized series is the more complete picture of the basin-wide story;
the observed series is what the raw measurements show directly. The dashboard
presents both side by side.

### Limitations to disclose

- **Stationarity assumption.** The normalization assumes each polygon's
  year-type response is stationary — a Critical year in 2008 draws the polygon
  down at the same rate as a Critical year in 2014. Reasonable for short
  windows; may not hold over multi-decade windows if pumping or recharge
  sources shift.
- **Small-sample noise on late-baseline polygons.** Polygons with short records
  have noisier per-type rates; `23N01E07H001M` (baselines 2019) has only 6
  transition years across 4 of the 5 year types.
- **No spatial correction.** The normalization adjusts each polygon
  individually; it doesn't reconcile across polygons (a recovering area can't
  "lend" surplus to a draining area). Lateral connectivity isn't modeled here.

## Reading the dashboard

### Map color scheme

Polygons are colored by their **average normalized storage rate (AF/yr)** — the
year-type-weighted rate that corrects for late-baseline drag. Five continuous
bands, **upper-bound inclusive** (a value of exactly −100 falls in light green,
exactly −250 in light yellow, exactly −750 in orange, exactly 0 in medium
green):

| Rate (AF/yr) | Color        | Meaning                  |
|--------------|--------------|--------------------------|
| > 0          | dark green   | gaining storage          |
| 0 to −100    | medium green | very slight loss         |
| −100 to −250 | light green  | modest loss              |
| −250 to −750 | light yellow | meaningful loss          |
| < −750       | orange       | fastest-draining cells   |

### Section labels

When the **section-label** layer is on, each polygon shows **two values**: the
polygon's short section label (or "Chico" for the aggregate) on the first line,
and its **average normalized storage rate in AF/yr** on the second. Labels sit
in a small high-contrast chip so both values stay legible across the full color
range.

### Polygon popup

Click any polygon for the full per-polygon detail, including:

- **Specific yield with its borehole provenance** — the Sy value plus how many
  boreholes inside the polygon back it (and how many have ≥200 ft of logged
  lithology), reinforcing that Sy is measured, not assigned. Fallback polygons
  say so explicitly.
- **Starting level**, **ending level**, and **change in level** for that
  polygon — in ft msl (elevation) by default, or in ft BGS (depth below ground)
  when the toolbar's *"Show popup levels as depth (ft BGS)"* toggle is on.
- The **storage-change formula with the actual values plugged in** —
  `change in elevation × polygon area × Sy` (or `−(change in depth) × area × Sy`
  in BGS mode) — so the cumulative change-in-storage result can be traced by
  hand.
- The polygon's **observed average rate** (cumulative ÷ span) and its
  **average normalized storage rate (AF/yr)** — the latter being the value used
  to color it on the map.
- **How the normalized rate is built** — a small table showing the polygon's
  own rate within each water-year type applied across the basin's full 26-year
  type mix. For late-baseline polygons this makes explicit why the normalized
  rate differs from a naive "observed cumulative ÷ 26." For example,
  `23N01E07H001M` (record starts 2019) has an observed cumulative of −2,630 AF;
  spread naively over 26 years that is only ≈ −101 AF/yr, but its
  drought-year-weighted normalized rate is **−371 AF/yr**, because its record
  misses the pre-2019 drawdown that the year-type weighting restores.

Basemaps (CartoDB Positron, Esri World Topo, Esri World Imagery satellite, and
OpenStreetMap) can be toggled under the map; the proposed 2027 RMS well
locations are shown as small dots (10 for the Chico aggregate, one for each
other polygon).

## What's in this repo

| File                                      | Purpose                                                         |
|-------------------------------------------|-----------------------------------------------------------------|
| `index.html`                              | The dashboard — single-file HTML, ready to push to GitHub Pages |
| `scripts/build_sy_svsim.py`               | Recomputes polygon Sy from DWR's SVSim Texture Data             |
| `scripts/build_dashboard.py`              | Main analysis — reads polygons + measurements, computes storage |
| `scripts/build_html.py`                   | Single-file HTML/CSS/JS template (called by build_dashboard.py) |
| `data/polygon_sy_svsim_three_zone.csv`    | Polygon-by-polygon Sy values (output of build_sy_svsim.py)      |
| `data/condition_analysis_three_zone.json` | Per-polygon storage attribution by SVI water-year type          |
| `data/sustainability_2042_three_zone.json`| Per-polygon and basin totals (observed + normalized)            |
| `data/basin_annual_three_zone.json`       | Basin-wide gap-attributed annual ΔStorage (observed + normalized) |
| `data/model_data_three_zone.json`         | Per-polygon annual GWE + storage time series                    |
| `data/polygon_storage_2025_three_zone.csv`| Per-polygon WY 2025 detail (csv)                                |
| `data/storage_timeseries_three_zone.csv`  | Annual basin cumulative storage 1999–2025 (csv)                 |
| `data/polygon_map_three_zone.svg`         | Static polygon map (colored by normalized rate)                 |
| `data/basin_buckets_chart_three_zone.svg` | Bar chart — basin storage by SVI year type                      |
| `data/basin_cumulative_chart_three_zone.svg` | Time series — basin cumulative ΔStorage with SVI bands        |
| `data/storage_context_three_zone.svg`     | Proportion view (16 MAF total vs cumulative deficit)            |

## Reproducing

```bash
# 1. (one-time) Python deps
pip3 install --user pyproj shapely markdown

# 2. (one-time per Sy refresh) recompute polygon Sy via SVSim
python3 scripts/build_sy_svsim.py
# downloads /tmp/svsim/svsim_texture_data.csv (~9 MB) on first run,
# writes data/polygon_sy_svsim_three_zone.csv

# 3. Build the dashboard
python3 scripts/build_dashboard.py
# reads polygons, wells, and measurements from ../2027-BC-prop-network/,
# reads the three-zone Sy CSV, writes the data outputs and single-file index.html
```

Both scripts read polygons + wells + measurements from the sibling
[`2027-BC-prop-network`](https://github.com/agubc-vina/2027-BC-prop-network)
repo (`../2027-BC-prop-network/js/*.js`). The embedded methodology section in
`index.html` is this README, rendered at build time via the `markdown`
package.

## Live dashboard

Published via GitHub Pages at
<https://agubc-vina.github.io/2027-BC-Storage/>. The interactive briefing in
`index.html` is single-file (all SVGs and JS inlined), so there are no asset
paths to fix up after deploy. Click any polygon on the map for per-polygon
detail (Sy, elevations, the storage-change calculation, and the normalized
rate).

## Status

Independent analysis prepared by AGUBC for internal discussion with Vina GSA
technical staff and AGUBC Board members. Comments and corrections welcomed.
