# Ventura County Political History, 1876–2026

An election-history dashboard, a candidate explorer and an auditable research database built from county election returns. It covers 92 contests, 366 candidate results and 23 offices, with 38 presidential elections (1876–2024) and 16 gubernatorial elections (1962–2022).

The project keeps three things apart that are usually mixed together: what the data says, how much to trust it, and what is still missing.

## What is in the repo

| Path | What it is |
|---|---|
| `dashboard.html` | Single-file interactive dashboard (open in any browser; loads Chart.js from cdnjs) |
| `paper/ventura_county_political_history_paper.pdf` | 8-page research paper with models and diagrams |
| `data/ventura_politics.sqlite` | The research database (6 tables, 3 views) |
| `data/*.csv` | The same tables as CSV, plus the model input series |
| `data/raw/` | The three supplied files in CSV and XLSX, untouched |
| `figures/` | Seven figures used in the paper |
| `scripts/` | Everything needed to rebuild all of the above |

## The dashboard

1. **Election history.** Pick President or Governor, then any election year. See the two-party margin for every election, the Democratic share over time, the winner, candidate shares and any data-quality notes for that election. Clicking a bar opens that election.
2. **Candidate explorer.** Search by name and filter by office, party, stage and year. Sort by year, votes or share. A single-name search summarises that candidate’s contests.
3. **Research record.** Data-quality issues, missing-data records, sources and coverage by office, all in one place.

## The research database

```
sources ──< contests ──< results >── candidates
data_quality_issues     missing_data
```

| Table | Rows | Purpose |
|---|---|---|
| `contests` | 92 | One row per contest: date, stage, office, district, total votes, source, review status |
| `results` | 366 | One row per candidate in a contest: party, votes, share, winner flag |
| `candidates` | 295 | Normalised names. Pooled “Other” rows are not treated as a person |
| `sources` | 5 | Provenance. URLs were not supplied, so every source is marked unverified |
| `data_quality_issues` | 16 | Each problem with evidence and a suggested action |
| `missing_data` | 15 | Offices and periods not yet in the database, with priority |

Views: `v_winners`, `v_candidate_history`, `v_coverage`.

```sql
-- Every contest a candidate entered
SELECT year, stage, office, party, votes, vote_pct, winner
FROM v_candidate_history WHERE candidate = 'Ronald Reagan' ORDER BY year;
```

Review status per contest is one of `carried_over_unverified`, `extract_loaded_unverified`, `needs_source_reconciliation` or `incomplete`. Nothing is marked verified yet.

## Findings

- A change-point model finds **five regimes** in the presidential margin: Republican (1876–1916), Republican landslides (1920–1928), Democratic (1932–1964), Republican (1968–1988) and Democratic (1992–2024, mean D+8.2 points).
- The winning party changed **9 times** across 38 elections. Eight elections were decided by under five points; four of them are 1992–2004.
- Governor results are **only loosely tied** to the previous presidential vote (R² = 0.19, n = 16, leave-one-out error 8.3 points). A straight-line forecast of 2026 is not usable and the paper says why.
- Presidential turnout grew about 42% per decade from 1932, peaked at 422,825 in 2020 and fell to 387,595 in 2024.

![Presidential margin by regime](figures/fig1_presidential_margin_regimes.png)

## Data-quality flags worth knowing

- **1912 president.** The analytics file labels the winner Democratic (+42 points). The cleaned file shows Theodore Roosevelt (Progressive) with 54.9%. This project uses the cleaned file and counts Roosevelt with the Republican lineage.
- **1962 governor.** The Democratic candidate is listed as Glenn M. Anderson, who was lieutenant governor; the Democratic nominee for governor was Pat Brown. This is flagged from historical knowledge, not from a source in the bundle. Check it against the county statement of vote.
- **2024 president.** Total votes differ by 109 between files.
- **Primaries.** “Winner” means top finisher, not nominee.
- **Incumbent** is empty in every row.
- **2026 general** (November 3, 2026) is not in the data; only the June 2 primary is.

The full list is in `data/data_quality_issues.csv` and on the dashboard’s Research record tab.

## Rebuild everything

```bash
pip install -r requirements.txt
python scripts/build_data.py        # CSV tables, SQLite, dashboard payload, issue and missing-data logs
python scripts/analysis.py          # models, model_results.json, figures
python scripts/build_dashboard.py   # dashboard.html from the template and payload
python scripts/make_paper.py        # paper PDF
```

To deploy the dashboard, publish `dashboard.html` with GitHub Pages or open it locally.

## Adding older and local elections

Priority list (details in `data/missing_data.csv`): gubernatorial generals 1879–1958, congressional and statewide generals before 2026, county offices before 2020, city councils, school districts, special districts, ballot measures and registration history.

For each new contest:

1. Append rows to the raw file with a source URL and retrieval date.
2. Add the number of seats for multi-seat races and fill `incumbent`.
3. Run `build_data.py`. Anything that does not reconcile to the official contest total stays `needs_source_reconciliation`.
4. Move a contest to verified only after reconciling it against the official statement of vote.

## Limits

Sixteen gubernatorial observations support only coarse inference. Two-party shares ignore third parties. All values come from supplied files that have not been checked against official statements of vote. There are no precinct, city or turnout-rate data.

## Sources named in the supplied files

Wikimedia Commons Ventura County presidential returns; a Ventura County gubernatorial results table; California Secretary of State Statement of Vote for the June 2, 2026 primary; California Secretary of State Report of Registration, May 18, 2026. Links were not supplied.
