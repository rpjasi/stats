# 04 — Delivery, Validation & Governance

> **Stack:** SQL Managed Instance · Azure · PyCharm · Git · AI agents · Qlik Cloud
> **Index:** [00 - Analytics Programme Index](./00%20-%20Analytics%20Programme%20Index.md)

---

## 🔁 The Shape of the Pipeline

```
SQL MI (policy, claims, bordereaux)
   └─ nightly snapshot → Analytics.RenewalOpportunity     [SQL]
        └─ feature build + training                        [Python, PyCharm, Git]
             └─ model artefact + metrics                   [versioned, Azure ML or blob]
                  └─ scoring job → Analytics.RenewalScore  [Python, scheduled on Azure]
                       └─ walk + simulation → Analytics.ForecastWalk
                            └─ Qlik Cloud apps             [read-only consumption]
```

One principle carries the whole design: **SQL MI is the integration contract.** Python writes scores to tables; Qlik reads tables. Neither side needs to know the other exists. This keeps Qlik simple, makes the scores reusable by anything else in the business, and means you can swap the modelling layer without touching the BI layer.

---

## 🗄️ SQL Managed Instance

- **Schemas:** `Analytics` for outputs, `Staging` for intermediate, source schemas untouched.
- **Snapshots:** temporal (system-versioned) tables where you control the schema; otherwise a nightly insert-only snapshot. Reconstructing history from mutable tables is the number-one source of leakage.
- **Feature logic in SQL, not Python**, where it's set-based — joins, aggregates, windowed history. Push compute to the database; pull a modelling-ready frame into Python.
- **Views for contracts:** Qlik binds to views (`Analytics.vw_RenewalScore`), not tables. You can then refactor the physical table without breaking dashboards.
- **Reconciliation view** that ties `sum(expiring_gwp)` back to the bordereaux each month — a broken tie is your earliest warning that something upstream changed.

---

## 🐍 Python / PyCharm / Git

Repo layout — keep the production path notebook-free:

```
retention-forecast/
├── src/retention/
│   ├── data/          # SQL extraction, snapshot loaders
│   ├── features/      # transformations, encoders
│   ├── models/        # fit, predict, calibrate
│   ├── forecast/      # walk, simulation, reconciliation
│   └── evaluation/    # backtest, calibration, A/E reports
├── conf/              # YAML: model params, schemes, scenarios, dates
├── notebooks/         # EDA only, outputs cleared on commit
├── tests/
├── pipelines/         # entry points: train.py, score.py, forecast.py
└── CLAUDE.md          # conventions, schema notes, agent context
```

- **Config-driven.** Scenarios, rate plans, knot positions, date windows in YAML. Nobody should edit Python to run a +12% scenario.
- **Determinism.** Seed everything; log the seed, git SHA and config hash with every model artefact. You will be asked "why is this number different from last week".
- **Tests that actually matter here:** leakage tests (assert no feature has a timestamp after `snapshot_date`), reconciliation tests (walk lines sum to the total), schema contract tests on the SQL views. Unit-testing a sklearn wrapper is less valuable than either.
- **Pre-commit:** `ruff`, `black`, `nbstripout`. Branch per piece of work, PR into main, no direct commits.
- **Secrets:** never in the repo. Azure Key Vault + managed identity; local dev via `.env` that is gitignored.

---

## ☁️ Azure

Start deliberately boring. You need a scheduled Python job that reads SQL MI and writes back.

| Need | Minimal option | Scale-up option |
|---|---|---|
| Orchestration | Azure Data Factory pipeline calling a container | Azure ML pipelines, or Airflow if the team grows |
| Compute | Azure Container Apps job / Container Instances on a schedule | Azure ML compute cluster |
| Model registry | Blob storage + a `ModelRegistry` table in SQL MI | Azure ML model registry with MLflow |
| Secrets | Key Vault + managed identity | same |
| Monitoring | Log Analytics + a SQL-based metrics table | Azure Monitor alerts |

Do not stand up Databricks or a feature store for this. The data is small, the cadence is monthly-to-weekly, and complexity here buys nothing except cost and a maintenance burden on a one-person team.

**Networking caveat:** SQL MI usually sits in a VNet. Whatever compute you choose must be VNet-integrated — check this in week 1, it's the most common thing that blocks phase 3 unexpectedly.

---

## 🤖 AI Agents in the Workflow

Where they pay off, and where they don't:

| Use | Verdict |
|---|---|
| Scaffolding repo structure, boilerplate, config plumbing | ✅ Strong |
| Translating business logic between SQL and pandas | ✅ Strong |
| Writing tests, docstrings, type hints | ✅ Strong |
| Explaining unfamiliar library APIs, drafting Qlik expressions | ✅ Good |
| First-pass code review, spotting leakage patterns | ✅ Good, with human confirmation |
| Choosing the model, setting the target definition, judging calibration | ❌ You own this |
| Anything touching the metric definitions Finance signed off | ❌ Human-only |

Practical setup: keep a `CLAUDE.md` in the repo with the schema, the agreed metric definitions, and the house conventions. Agents that know your `RenewalOpportunity` grain and your retention definition are far more useful than ones guessing. Treat that file as a living artefact — every time you correct an agent on the same thing twice, it belongs in there.

Guardrail: agent-written SQL touching production data gets read line by line. An agent that quietly joins on the wrong date grain produces plausible-looking numbers, which is worse than an error.

---

## 📈 Monitoring in Production

| Check | Cadence | Trigger |
|---|---|---|
| Actual vs expected renewals (count and premium) | Monthly | A/E outside 95% band two months running |
| A/E by scheme and broker | Monthly | Any material segment out by >5% |
| Calibration curve on the last 12 months | Quarterly | Visible drift from diagonal |
| Feature drift (PSI on key features) | Monthly | PSI > 0.25 on any strong feature |
| Score coverage | Weekly | Any expiring policy without a score |
| Forecast vs actual GWP | Monthly | Cumulative bias > 3% |
| Full refit | Annually, pre-budget | — |

Write these to a `Analytics.ModelMonitoring` table and put them in the retention monitor app. Monitoring that lives in a notebook nobody opens is not monitoring.

---

## 📋 Model Governance

Keep a model card per model, in the repo, updated at each refit:

- Purpose and permitted use (and explicitly: **not** permitted uses)
- Target definition, population, exclusions
- Features, with data lineage
- Training window, performance, calibration, backtest results
- Known limitations and segments where it underperforms
- Owner, approver, review date

Sign-off should involve Finance (definitions tie to the budget), Underwriting (elasticity is plausible), and Compliance (see below).

---

## ⚖️ Regulatory Constraints — UK MGA

Flagging these early because they constrain *how the model may be used*, not just how it's built.

- **FCA pricing practices (PS21/5 / ICOBS 6B)** — for **home and motor personal lines**, the renewal price offered to an existing customer must be no higher than the equivalent new business price. A retention model must therefore **not** be used to price-discriminate at renewal on the basis of predicted inertia in those lines. Using it to prioritise *service and contact* is fine; using it to set *price* in scope products is not. Confirm with Compliance which of your products are in scope.
- **Consumer Duty** — where the book is retail, models influencing customer outcomes fall under fair value and outcomes monitoring. Segment-level outcome testing is expected, and your A/E-by-segment reporting largely covers it.
- **Delegated authority** — capacity providers may have contractual rights over analytics on their data, and audit rights. Check binder terms before shipping scheme-level analytics externally.
- **Model risk / SS1/23 proportionality** — full model risk management is a PRA expectation on banks, but the principles (documentation, validation independence, ongoing monitoring) are a sensible bar and increasingly asked about in due diligence.
- **GDPR** — commercial policy data on sole traders and partnerships is personal data. Document lawful basis and retention; keep the analytics environment inside your existing data map rather than creating a shadow copy.

None of this blocks the work. It shapes the framing: **prioritisation and forecasting, not renewal price optimisation.** Say that explicitly in the model card, and the compliance conversation becomes short.

---

## ✅ Phase Gate Checklist

Don't move to the next phase until:

**Phase 0 → 1** — retention definition signed off in writing · fact table reconciles to bordereaux · leakage tests passing · 3+ years of snapshots or a documented reason why not

**Phase 1 → 2** — beats flat-rate benchmark on a held-out year · calibrated within ±3pp across deciles · no segment A/E out by >5% · elasticity curve reviewed and judged plausible by underwriting

**Phase 2 → 3** — walk reconciles line by line · backtest beats last year's human budget · P10/P90 band judged plausible (not implausibly narrow) · scenarios reproducible from config

**Phase 3 → run** — scores land in SQL MI on schedule · Qlik apps signed off · monitoring table populating · handover doc and model card complete

---

*[Index](./00%20-%20Analytics%20Programme%20Index.md) | Prev: [03 - Methods - Growth and Budget Forecasting](./03%20-%20Methods%20-%20Growth%20and%20Budget%20Forecasting.md)*
