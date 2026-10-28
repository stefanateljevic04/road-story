# U.S. Road Deaths — Fifty Years of Traffic Fatalities, 1975–2024

Two public-domain U.S. federal datasets, assembled for a scrollytelling lab about
telling an honest data story.

- `us_road_deaths_by_year.csv` — **50 rows, one per year (1975–2024), 5 columns.** The spine.
- `fatal_crashes_2022_when.csv` — **43 rows**, the *within a single year* breakdown (day-of-week,
  hour, month) of 2022 deaths. For the stretch goal only.

**Sources.** Fatalities and the official fatality rate come from **NHTSA's Fatality Analysis
Reporting System (FARS)** — a *census* of every U.S. fatal traffic crash, not a survey — via
NHTSA's master table *"Motor Vehicle Traffic Fatalities and Fatality Rates, 1899–2024"* (Annual
Report Portal, cdan.dot.gov). Vehicle-miles traveled come from the **Federal Highway
Administration (FHWA)**. Population is the **U.S. Census** resident-population series. The
within-year file is derived from the FARS 2022 crash-level national file. All are U.S.
Government works: **public domain.**

> **These are fatal crashes only.** FARS counts *deaths*. This data can support true statements
> about road *deaths*; it says **nothing** about total crashes, injuries, or near-misses — that
> data isn't here. Don't let a chart imply otherwise.

## `us_road_deaths_by_year.csv` — columns

| Column | Type | Range (this file) | Notes |
| --- | --- | --- | --- |
| `year` | int | 1975 … 2024 | Calendar year. 50 rows, no gaps. |
| `fatalities` | int | 32,479 (2011) … 51,093 (1979) | People killed in U.S. motor-vehicle traffic crashes that year. |
| `vmt_billions` | float | 1,327.7 (1975) … 3,294.0 (2024) | Vehicle-miles traveled, in **billions** of miles. The **exposure** — how much driving happened. |
| `rate_per_100m_vmt` | float | 1.08 (2014) … 3.35 (1975/80) | Deaths per **100 million** vehicle-miles = `fatalities / (vmt_billions × 10)`. **Risk per unit of driving.** |
| `population` | int | ~216.0M (1975) … ~340.1M (2024) | U.S. resident population, for a per-capita cross-check. |

Every value loads as a **string** — convert all five columns to numbers on load.

## `fatal_crashes_2022_when.csv` — columns (stretch goal)

| Column | Type | Values | Notes |
| --- | --- | --- | --- |
| `dimension` | string | `day_of_week`, `hour`, `month` | Which breakdown this row belongs to. |
| `bucket` | string | `Sunday`…`Saturday`; `00`…`23`; `January`…`December` | The category. |
| `sort_key` | int | 1–7 / 0–23 / 1–12 | Order buckets *within* a dimension (don't sort by `deaths`). |
| `deaths` | int | 1,075 … 7,717 | 2022 fatalities in that bucket. |

The three dimensions each sum to the same 2022 total (**42,721**); they're three cuts of one year.

## Quirks that matter for *judging your story*

- **Count and rate tell different stories — this is the whole point.** `fatalities` is a raw
  **count**; `rate_per_100m_vmt` divides it by how much everyone drove. From 1975 the *rate*
  fell by roughly two-thirds (3.35 → ~1.1) — a genuine safety triumph — while the raw *count*
  wandered up and down, because driving more than doubled. **"Roads got safer"** is a claim about
  the **rate**; **"more people are dying"** is a claim about the **count**; both are true at once.
  Every chart and sentence must make clear which one it shows.
- **2020 breaks every tidy narrative.** From 2019 to 2020, miles driven **fell** ~11%
  (3,261.8 → 2,903.6 B) during the pandemic — yet deaths **rose** ~7% (36,355 → 39,007) and the
  rate **jumped** from 1.11 to 1.34, its sharpest rise in decades. Any story that says "safety
  improved every year" or "2020 was just fewer cars on the road" is contradicted by this row.
  This is where you prove your story is honest.
- **Recent years get revised — the CSV may not match the headline you remember.** FARS figures
  pass through *Early Estimate → Annual Report File (ARF) → Final*, and the numbers move. The
  headlines were **2020 = 38,824**, **2022 = 42,514**, **2023 = 40,901**; the FARS **Final**
  values in this file are **39,007**, **42,721**, and **41,025**. NHTSA itself notes 2022 was
  "updated from 42,514." **2024 (39,254) is a preliminary Annual Report File estimate, not Final.**
  If your source disagrees with the CSV by a few hundred, this is why — say so, don't hide it.
- **The `population` column splices Census vintages.** It's stitched from four Census
  release vintages, so there are small seams (most visibly 1999→2000, and a large upward
  migration revision for 2021–24). It's fine for per-capita *sanity checks*; don't build a
  precise year-over-year population-change story on it.
- **Within-year counts are exposure-blind.** In `fatal_crashes_2022_when.csv`, deaths peak in
  the **evening** and on **Saturday** — but that's partly because that's when the most people are
  driving. Raw counts measure *when deaths happen*, not *when driving is most dangerous per mile*;
  the latter needs miles-driven-by-hour, which isn't in this file. Don't call 6pm "the most
  dangerous hour" on this data alone.

## The story this data tells

Plot the **rate** and you see one of the great public-health wins of the last century — seatbelts,
airbags, crumple zones, drunk-driving enforcement, safer roads — cutting the death rate by more
than half even as Americans drove ever more. Plot the raw **count** and the win is muddier, because
growth in driving pushes deaths up even as each mile gets safer. Then **2020** ruptures both lines
at once. Which number you choose to plot *is* the argument — so choose it honestly, and check every
sentence against these columns.
