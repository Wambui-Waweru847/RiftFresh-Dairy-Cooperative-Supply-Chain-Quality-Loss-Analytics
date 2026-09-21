# RiftFresh-Dairy-Cooperative-Supply-Chain-Quality-Loss-Analytics


A 2-page Power BI report diagnosing where a dairy cooperative is losing milk and revenue across its collection network, and prioritizing where to intervene first — built on a 65,000-row star schema.

## Business Problem

RiftFresh collects milk from farmers across four counties through a network of collection centers, but a meaningful share of what's collected never reaches revenue — rejected on quality grounds, lost to poor refrigeration, or delayed past acceptable collection windows. Leadership needed a way to see, at a glance, which counties and collection centers are driving that loss, what it's costing in KES, and which interventions would recover the most value for the investment required.

## Report Structure

1. **RiftFresh Dairy Cooperative Dashboard — "Where are we making the losses"**
   Headline KPIs (Milk Collected, Milk Rejected, Revenue, Revenue Loss) with period-over-period deltas, a Revenue Loss trend by month, Milk Rejected by County, and a ranked table of the five collection centers most in need of operational intervention.
2. **Accountability and Prioritization — "What should we do?"**
   Critical-tier loss summary (KES lost, average delay, % poor refrigeration, % remote route), a Top Actions table ranking interventions by investment, annual recovery, payback period, effort, and impact, a county-level Dairy Loss Priority List, and a summary recommendation.

## Why This Project

Raw inbound supply data from a dairy cooperative doesn't arrive analysis-ready — deliveries, rejections, and loss reasons sit in flat operational records with no built-in way to compare periods, rank severity, or connect a loss figure to a specific recommended action. This project turns that raw data into a governed semantic model that a cooperative manager can actually act on: not just "here's how much we lost," but which five collection centers to fix first, and what each fix is worth.

## Architecture

The model follows a standard star schema pipeline — a single fact table at the center, surrounded by conformed dimensions, with a disconnected table layered on top purely to drive time-intelligence measures:

```
Raw Supply, Rejection & Loss Records
        |
        ▼
   Power Query (M)
   cleaning, type casting, merges
        |
        ▼
      Fact_Supply
        ├── Dim_Farmer
        │       └── Farmer Risk Tier (calculated column)
        ├── Dim_CollectionCenter
        │       └── County (hierarchy level)
        └── Dim_Date
                └── Time Period Table (disconnected — powers YTD/PM/3M/6M/12M switch)
```

This design keeps each piece doing one job: Power Query handles shaping, the star schema handles relationships, the calculated column handles risk classification, and the disconnected table handles the period selector on both report pages — none of them overlapping.

## Data Model

![RiftFresh ERD](<img width="494" height="411" alt="Screenshot 2026-09-21 120801" src="https://github.com/user-attachments/assets/a4a40ff4-5893-4e03-a114-3c5b2f1538e9" />
)

- Star schema: 1 fact table (`Fact_Supply`), dimension tables for Farmer, Collection Center (rolling up to County), and Date
- 65,000 rows in the core fact table
- Disconnected Time Period Table driving the YTD / PM / 3M / 6M / 12M selector visible on both report pages
- `Farmer Risk Tier` calculated column on `Dim_Farmer`, rolling up to county-level Critical Farmer counts
- Loss reasons (poor refrigeration, remote route, delay) modeled as attributes on the fact grain, enabling the % breakdowns on the Accountability page



## Key Techniques

- **DAX** — dynamic period comparisons (SELECTEDVALUE/SWITCH over the disconnected Time Period Table), tier-based aggregations for Critical Farmers and Critical Loss KES, conditional formatting measures driving the loss-rate and refrigeration progress bars, and a dynamic recommendation label (Immediate Intervention / Quality Audit) based on loss-rate thresholds
- **Power Query** — standardizing collection center and county naming, calculating loss rate and rejection rate columns, handling blanks/error rows in delay and refrigeration fields
- **Manually-maintained `PriorityActions` table** — Action, Category, Investment, Annual Recovery, Payback, Effort and Impact scores, driving the Top Actions Ranked by Estimated Impact table on page 2

## Tech Stack

- Power BI Desktop (DAX, Power Query/M)


## Screenshots

![Executive Overview — Where are we making the losses](screenshots/page1_executive_overview.png)
![Accountability and Prioritization — What should we do?](screenshots/page2_accountability.png)

## Repo Contents

- `RiftFresh_Dairy_Report.pbix` — the full Power BI file
- `scripts/dax_measures.dax` — all DAX measures, exported as text for readability/version tracking
- `scripts/power_query_m.pq` — Power Query M code for key transformation steps
- `docs/requirements.md` — business & functional requirements this report was built to satisfy
- `docs/images/erd.png` — entity-relationship diagram of the star schema

## Notes

This report uses synthetic/portfolio data and does not represent a real dairy cooperative's operations.
