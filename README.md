# PSX-Sector-Risk-Analysis
This project builds a risk-adjusted, macro-aware ranking of Pakistan Stock Exchange (PSX) sectors, so that retail investors, junior advisors, and fintech teams can see which sectors deliver reliable returns versus which only look attractive because of short-term volatility or a temporary macro tailwind

# MP1 — Sector Return & Risk Analysis

**AuratTech Data & AI Fellowship — PSX Capstone Project**

This milestone computes daily returns per ticker, aggregates them to the sector level, and ranks the six PSX sectors by average return and risk (standard deviation).

## Prerequisites

Run these against the tables created in the schema setup step:

- `sector_prices` — `date, ticker, sector, open, high, low, close, volume, is_anomaly`
- `policy_events` — `event_date, old_rate, new_rate, direction, change_bps, notes, verified`

See [`schema_create.sql`](./schema_create.sql).

## Queries

All queries live in [`mp1_return_risk_queries.sql`](./mp1_return_risk_queries.sql). Run them in order — Q1 creates a view that every later query depends on.

| # | Query | Purpose |
|---|---|---|
| Q1 | `daily_returns` view | Computes `(close - prev_close) / prev_close` per ticker using `LAG()` |
| Q2 | Avg return per ticker | Per-ticker granularity, before trusting sector aggregates |
| Q3 | Std dev per ticker | Per-ticker risk |
| Q4 | Avg return per sector | Sector-level return |
| Q5 | Std dev per sector | Sector-level risk |
| Q6 | Annualized return & volatility | Scales daily stats by 252 trading days/year |
| Q7 | Rank sectors by return | `RANK() OVER (ORDER BY avg_return DESC)` |
| Q8 | Rank sectors by risk | Lowest std dev ranked safest |
| Q9 | Combined return + risk ranking | Return rank and risk rank side by side |
| Q10 | Risk-adjusted return | Return per unit of risk (Sharpe-style, rf = 0) — the headline comparison metric |

## Notes

- `WHERE is_anomaly = FALSE` filters out flagged bad data points before they skew the averages. Double-check this flag is actually populated for the Autos and Oil & Gas sectors — those were pulled from Excel workbooks that didn't originally have an `is_anomaly` column, so it currently defaults to `FALSE` for every row.
- Q9/Q10 are the direct answer to "rank by return and risk" — Q9 for a side-by-side view, Q10 for the single risk-adjusted number.
- Next milestone: join `daily_returns` against `policy_events` to compare sector performance in the days following a rate cut/hike vs. a hold.

## How to run

```bash
psql -d your_database -f schema_create.sql
\copy sector_prices FROM 'all_sector_prices.csv' CSV HEADER
\copy policy_events FROM 'sbp_policy_events_corrected.csv' CSV HEADER   
psql -d your_database -f mp1_return_risk_queries.sql
```
https://docs.google.com/spreadsheets/d/1OPQrXDcjTX3TirGmFW1os4ISlZ31GGrUgLFzSHY9Qf0/edit?gid=1407734078#gid=1407734078
