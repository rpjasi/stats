# 🏢 Work — Statistical Analytics Programme

**Owner:** Renato Jasi
**Context:** Insurance MGA — budget, policy retention and growth forecasting
**Started:** 2026-08-17
**Phase 1 scope:** Policy-level retention modelling + GWP growth forecast feeding the budget

---

## 🗺️ Note Navigation

| # | Note | Purpose |
|---|---|---|
| 00b | [**Start Simple**](./00b%20-%20Start%20Simple.md) | ⭐ Read first — the escalation ladder from spreadsheet to full programme |
| 01 | [Phase 1 Approach](./01%20-%20Phase%201%20Approach.md) | Framing, definitions, the renewal walk, phasing |
| 02 | [Methods — Retention Modelling](./02%20-%20Methods%20-%20Retention%20Modelling.md) | Model ladder, features, pitfalls |
| 03 | [Methods — Growth & Budget Forecasting](./03%20-%20Methods%20-%20Growth%20and%20Budget%20Forecasting.md) | Bottom-up walk, new business, uncertainty |
| 04 | [Delivery, Validation & Governance](./04%20-%20Delivery%20Validation%20and%20Governance.md) | Stack, backtesting, monitoring, regulatory |

Study vault: [00 - Index](../00%20-%20Index.md)

---

## 🎯 The One-Sentence Framing

> Forecast next year's GWP by predicting, for every policy that expires, whether it renews and at what premium — then add a separately modelled new-business stream, and express the whole thing as a bridge that Finance can argue with line by line.

Everything else in these notes is detail hanging off that sentence.

---

## 🧱 The Core Identity

Write this on a whiteboard before writing any code. Every model in the programme is estimating one term of it.

$$
\text{GWP}_{t+1} = \underbrace{E_t \cdot r \cdot (1+\rho) \cdot (1+\epsilon)}_{\text{renewal book}} + \underbrace{N_{t+1}}_{\text{new business}} + \underbrace{M_{t+1}}_{\text{MTAs}} - \underbrace{C_{t+1}}_{\text{cancellations}}
$$

| Symbol | Meaning | How it's estimated |
|---|---|---|
| $E_t$ | Expiring GWP available to renew | Deterministic — from the policy system |
| $r$ | Retention rate (premium-weighted) | **Modelled** — [Note 02](./02%20-%20Methods%20-%20Retention%20Modelling.md) |
| $\rho$ | Average rate change applied at renewal | Underwriting input / scenario lever |
| $\epsilon$ | Exposure change (sums insured, turnover, headcount) | Trend + index |
| $N_{t+1}$ | New business GWP | **Modelled** — [Note 03](./03%20-%20Methods%20-%20Growth%20and%20Budget%20Forecasting.md) |
| $M, C$ | Mid-term adjustments, cancellations | Seasonal ratio to base, or hazard model |

Note $r$ and $\rho$ are **not independent** — retention falls as rate change rises. Modelling that elasticity is the single highest-value output of the programme, because it turns the budget from a spreadsheet into a decision tool.

---

## 📅 Phasing at a Glance

| Phase | Weeks | Output | Success test |
|---|---|---|---|
| 0 — Foundation | 2–3 | Point-in-time renewal fact table; agreed metric definitions | Retention rate reconciles to Finance's number to ±0.5pp |
| 1 — Retention baseline | 4–6 | Calibrated logistic model + rate elasticity curve | Backtest: predicted vs actual renewals within ±2% at portfolio level |
| 2 — Growth & budget | 3–4 | Bottom-up walk, new business forecast, P10/P50/P90 | Prior-year backtest beats naive "last year + x%" |
| 3 — Delivery | 2–3 | Scores in SQL MI, three Qlik apps live | Account handlers use the renewal risk worklist unprompted |
| 4 — Run | ongoing | Monthly monitoring, drift alerts, annual refit | Actual-vs-predicted tracked monthly, no silent decay |

---

## ⚠️ The Five Things Most Likely to Kill This

1. **Definitional drift** — your retention rate won't match Finance's. Fix the definition in week 1, in writing, signed off. See [Note 01](./01%20-%20Phase%201%20Approach.md#-definitions-agree-these-before-any-modelling).
2. **Target leakage** — using fields updated at or after renewal. Snapshot discipline solves it. See [Note 02](./02%20-%20Methods%20-%20Retention%20Modelling.md#%EF%B8%8F-leakage-the-main-way-this-goes-wrong).
3. **Optimising AUC instead of calibration** — you are summing probabilities to get a GWP number. A well-ranked but badly calibrated model gives a wrong budget.
4. **Portfolio discontinuities** — scheme launches, binder losses, capacity changes, book transfers. History doesn't repeat, and these break naive time series.
5. **No route to action** — a model that only produces a forecast gets defunded. Ship the at-risk renewal worklist early.

---

## 🔗 Links Back to Theory

The ISLP course material maps directly onto this work:

- [Ch04 — Classification](../Chapters/Ch04%20-%20Classification.md) → logistic retention baseline
- [Ch05 — Resampling Methods](../Chapters/Ch05%20-%20Resampling%20Methods.md) → but replace k-fold with **time-based** splits here
- [Ch06 — Linear Model Selection](../Chapters/Ch06%20-%20Linear%20Model%20Selection.md) → regularisation for wide broker/scheme dummies
- [Ch07 — Moving Beyond Linearity](../Chapters/Ch07%20-%20Moving%20Beyond%20Linearity.md) → splines for the rate-change elasticity curve
- [Ch08 — Tree-Based Methods](../Chapters/Ch08%20-%20Tree-Based%20Methods.md) → GBM challenger model
- [Ch11 — Survival Analysis](../Chapters/Ch11%20-%20Survival%20Analysis.md) → mid-term cancellation and censored renewals

---

*Last updated: 2026-08-17*
