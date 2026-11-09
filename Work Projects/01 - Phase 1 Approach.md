# 01 — Phase 1 Approach

> **Scope:** Policy retention + GWP growth forecast, feeding the annual budget
> **Audience:** Technical — me and whoever picks this up
> **Index:** [00 - Analytics Programme Index](./00%20-%20Analytics%20Programme%20Index.md)

---

## 🎯 What Question Are We Actually Answering

Three distinct questions get bundled together as "forecasting". Separate them, because they need different methods and different consumers:

| Question | Who asks | Granularity | Method family |
|---|---|---|---|
| **Will this policy renew?** | Account handlers, underwriters | Policy | Classification |
| **What GWP will the book write next year?** | CFO, capacity providers | Portfolio / scheme | Aggregation + time series |
| **What happens if we push rate +8%?** | Underwriting director | Scheme | Elasticity / scenario |

The trap is building only the second one. Question 1 is where the operational value sits, and it is the *input* to question 2. Build 1 → aggregate to 2 → perturb for 3.

---

## 📏 Definitions — Agree These Before Any Modelling

This is not admin. Half of analytics projects in MGAs die here, because Finance, Underwriting and Ops each have a different retention number and nobody has written down why.

### Retention rate

Four legitimate variants. Pick and document:

$$
r_{\text{count}} = \frac{\#\text{renewed}}{\#\text{invited}}
\qquad
r_{\text{premium}} = \frac{\text{Renewed GWP}}{\text{Expiring GWP}}
$$

| Choice | Options | Recommendation |
|---|---|---|
| Denominator | Expiring policies vs *invited* policies | Model on **invited** (excludes policies we chose not to renew); report on **expiring** for Finance |
| Weighting | Policy count vs premium | Model count; **report premium-weighted** — it's what drives the budget |
| Premium basis | Actual renewal GWP vs expiring GWP at renewal | Use expiring GWP so retention isn't contaminated by rate change |
| Timing | Effective date vs quoted date | Effective date; measure with a lag window (see below) |

> [!warning] Premium-weighted retention using *actual* renewal premium double-counts rate change. If rate goes up 10% and every policy renews, "retention" reads 110%. Keep $r$ and $\rho$ separate in the walk.

### The measurement window

A policy expiring on 31 Dec isn't definitively lapsed until the grace/rebroke window closes. Define:

- **Renewal opportunity date** = expiry date of the prior term
- **Observation cut** = opportunity date + 60 days (tune to your book)
- Anything inside the cut at scoring time is **censored** — either exclude from training or handle with a discrete-time hazard model ([Ch11](../Chapters/Ch11%20-%20Survival%20Analysis.md))

### Exclusions to define explicitly

- Policies cancelled mid-term (never reached renewal) — separate model, not a retention lapse
- Non-invited / withdrawn (we declined, capacity withdrew, scheme closed) — exclude from $r$, handle as a deterministic adjustment
- Book transfers and novations — exclude, they distort every trend line
- Short-period, run-off and adjustable policies — flag, likely exclude from phase 1

---

## 🧮 The Renewal Walk (the deliverable Finance will actually read)

The forecast should be presented as a bridge, not a number:

```
Expiring GWP (known)                     12,400,000
  − Not invited / withdrawn                (310,000)
= Available to renew                     12,090,000
  × Retention rate           [modelled]        87.4%
= Retained at expiring rates             10,566,660
  × Rate change              [lever]           +6.0%
  × Exposure change          [indexed]         +1.5%
= Renewal GWP                            11,368,000
  + New business             [modelled]     2,150,000
  + Net MTAs                 [ratio]          340,000
  − Mid-term cancellations   [hazard]        (420,000)
= Forecast GWP                           13,438,000
```

Every line is either **known**, **modelled**, or a **lever**. Colour-code them in Qlik. When the number is wrong, this tells you which term was wrong — which is how the model earns trust in year two.

---

## 🗃️ Data Foundation (what Phase 0 must produce)

One table. Everything else is derived from it.

**`Analytics.RenewalOpportunity`** — one row per policy-term that reached (or will reach) a renewal decision.

| Group | Fields |
|---|---|
| Keys | `policy_id`, `term_id`, `opportunity_date`, `snapshot_date` |
| Target | `renewed_flag`, `renewal_term_id`, `days_to_decision` |
| Exposure | `expiring_gwp`, `renewal_gwp`, `sum_insured`, `exposure_measure` |
| Pricing | `rate_change_pct`, `technical_price`, `price_to_technical_ratio` |
| History | `tenure_years`, `prior_renewals`, `claims_count_3yr`, `claims_incurred_3yr`, `mta_count`, `days_late_last_payment` |
| Structure | `broker_id`, `scheme_id`, `product`, `capacity_provider`, `channel`, `region`, `payment_method`, `auto_renew_flag` |

### Point-in-time discipline

`snapshot_date` must be **before** the renewal decision — recommend `opportunity_date − 60 days`, matching when you'd actually score in production. Every feature is "as known at snapshot_date". Build this as a temporal query in SQL MI (system-versioned tables if available; otherwise a nightly snapshot table — do not reconstruct history from the live policy table, it's been overwritten).

> [!tip] The test for whether your fact table is right: can you regenerate the exact feature row for a policy that renewed in March 2024, as it would have looked in January 2024? If no, you have leakage.

### Data volume reality check

With one prior year you get one renewal cohort. That is enough for a logistic baseline but **not** enough for time series on new business, and not enough to separate a rate-elasticity effect from a market-cycle effect. Push for 3–5 years of snapshots in Phase 0. If only one year exists, say so explicitly in the model limitations and lean on cross-sectional variation (different brokers/schemes got different rate changes) rather than time variation.

---

## 🪜 Phasing in Detail

### Phase 0 — Foundation (2–3 weeks)

- Metric definitions agreed and signed off by Finance + Underwriting
- `RenewalOpportunity` table built and reconciled: sum of `expiring_gwp` must tie to the bordereaux
- EDA: retention by scheme, broker, tenure, rate-change band, premium size band
- Deliverable: a one-page data quality note listing every exclusion and why

### Phase 1 — Retention model (4–6 weeks)

- Logistic baseline → spline on rate change → GBM challenger
- Calibration is the acceptance criterion, not AUC
- Output: `p_renew` per policy + a rate-elasticity curve per scheme
- Deliverable: model card, backtest on held-out year, elasticity curves

### Phase 2 — Growth & budget (3–4 weeks)

- Aggregate policy scores → renewal walk
- New business funnel model
- Monte Carlo for P10/P50/P90
- Deliverable: budget forecast with scenario table, backtested against last year's actual

### Phase 3 — Delivery (2–3 weeks)

- Scoring job writes to SQL MI; Qlik apps built on top
- Three apps: retention monitor, budget bridge, at-risk renewal worklist
- Deliverable: live dashboards + handover doc

### Phase 4 — Run

- Monthly actual-vs-predicted, feature drift, annual refit
- See [Note 04](./04%20-%20Delivery%20Validation%20and%20Governance.md)

---

## 🧭 Design Principles

1. **Explainable beats accurate.** A GBM that beats logistic by 0.01 AUC but can't tell an underwriter *why* a renewal is at risk will lose the argument in the room. Ship the interpretable model first; use the GBM to prove you're not leaving much on the table.
2. **Aggregate up, don't model down.** Portfolio GWP forecasts built from policy-level predictions are auditable. Top-down forecasts fitted directly to portfolio GWP are not, and they can't answer scenario questions.
3. **Every model gets a naive benchmark.** Last year's retention rate applied flat. If you can't beat it, say so.
4. **Uncertainty is a deliverable.** A single point forecast will be treated as fact and then held against you. Ship the interval.

---

*[Index](./00%20-%20Analytics%20Programme%20Index.md) | Next: [02 - Methods - Retention Modelling](./02%20-%20Methods%20-%20Retention%20Modelling.md)*
