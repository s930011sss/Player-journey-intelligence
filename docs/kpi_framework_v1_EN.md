# KPI Framework & Business Questions v1
## Player Journey Intelligence Platform — Stage 0 Deliverable

> This document is the project's constitution: every dbt model, dashboard, and analysis
> must serve the questions defined here. Re-read it before starting each stage to stay on track.
> (Lives in the repo at `docs/kpi_framework.md`)

---

## 1. Business Thesis

How can a consumer products company combine two kinds of data — user **behavior**
(event streams) and user **voice** (reviews and community discussions) — to locate
friction points in the customer journey, and validate solutions through experimentation?

---

## 2. Five Questions the Gold Layer Must Answer

| # | Question | Serving mart | JD checkpoint |
|---|----------|--------------|---------------|
| Q1 | What are the conversion rates across the funnel (first visit → first add-to-cart → first purchase)? Where is the steepest drop-off, and how does it vary by category and traffic source? | `fct_funnel_daily` | Wise: funnel analysis |
| Q2 | What do weekly cohort retention curves look like? Is D7/D30 retention improving or deteriorating over time? | `fct_retention_cohort` | Wise: retention |
| Q3 | What share of dormant users reactivate? Which categories or behavioral traits predict successful win-back? | `fct_reactivation` | Wise: reactivation |
| Q4 | For the highest-churn categories, what is the structure of complaint themes in user voice data — and does it align with behavioral drop-off? | `fct_reviews_enriched` × funnel | Wise: diagnosis + Razer: GenAI |
| Q5 | For the biggest friction point found in Q1, does an A/B experiment support rolling out a candidate solution? | A/B notebook | Wise: A/B testing |

> Discipline rule: any table, chart, or feature that does not serve these five questions
> does not get built. It goes on the roadmap instead.

---

## 3. KPI Definitions (North Star + Pillar Metrics)

### North Star
- **Weekly Purchasers**: unique users with ≥1 purchase event in a given week.
  (Rationale: reflects acquisition, conversion, and retention health simultaneously,
  and is hard to inflate by optimizing any single funnel stage.)

### Funnel metrics (Q1)
| Metric | Definition | Notes |
|--------|------------|-------|
| Visit → Engage rate | Share of visiting users producing a second event within the same session | Session definition in §4 |
| Engage → Cart rate | Add-to-cart users ÷ engaged users | Bots excluded (rules in §5) |
| Cart → Purchase rate | Purchasing users ÷ add-to-cart users | User-day grain |
| End-to-end conversion | Purchasing users ÷ visiting users (28-day window) | Primary reporting metric |

### Retention & stickiness metrics (Q2)
| Metric | Definition |
|--------|------------|
| D7 / D30 retention | Share of a cohort with ≥1 event on day 7/30 (±1-day window) after first activity |
| Repeat purchase rate | Share of first-time buyers purchasing again within 30 days |
| Stickiness | DAU ÷ MAU (monthly) |

### Reactivation metrics (Q3)
| Metric | Definition |
|--------|------------|
| Dormancy rate | Share of existing users with no events for 28 consecutive days |
| Reactivation rate | Share of dormant users returning within the following 28 days |

### Voice metrics (Q4)
| Metric | Definition |
|--------|------------|
| Complaint intensity | Negative reviews ÷ total reviews, per category |
| Top complaint themes | Distribution of LLM-extracted `complaint_category` (per category, per month) |
| Voice-behavior gap score | Rank correlation between category complaint intensity and funnel drop-off |

### Guardrail metrics (for Q5's A/B test)
- Median event depth per session (protects against conversion gains at the cost of browsing quality)
- Median session duration
- (To be finalized during experiment design)

---

## 4. Key Grain & Definition Decisions (settle now, avoid fights in Stage 2)

1. **Session definition**: a new session starts when the gap between a user's consecutive
   events exceeds 30 minutes (industry convention; noted as tunable in README)
2. **Valid user**: ≥1 valid event within the observation window, after bot exclusion
3. **Dormancy**: 28 consecutive days with no events
4. **First-purchase attribution window**: 28 days from first visit
5. **Timezones**: store everything in UTC; convert at the analysis layer only

> Hidden interview checkpoint: half the gap between a senior and junior analyst is
> "who set these definitions, and why." Be ready to defend every line above.

---

## 5. Dirty Data Log (fill in after initial EDA)

| Issue | How discovered | Handling decision | Metrics affected |
|-------|----------------|-------------------|------------------|
| Bot traffic | (TBD: e.g., users with > N events/day) | (TBD) | All funnel metrics |
| Duplicate events | (TBD) | (TBD) | |
| Session boundary edge cases | (TBD) | (TBD) | |
| Timezone / timestamp anomalies | (TBD) | (TBD) | |
| (Add rows as discovered) | | | |

---

## 6. Stage 0 Exit Checklist

- [ ] Both datasets downloaded; one EDA pass completed on each
      (row counts, date range, column distributions, null rates)
- [ ] Dirty data log (§5) has at least 4 completed rows
- [ ] Repo scaffolded: `README.md` / `docs/` / `ingestion/` / `dbt_project/` / `analysis/` / `eval/`
- [ ] Environment works: DuckDB can read the event data and run `SELECT COUNT(*)`
- [ ] `dbt init` done; `dbt debug` passes (DuckDB adapter)
- [ ] pre-commit + sqlfluff installed
- [ ] This document committed to `docs/` as one of the project's first commits
