# 03 — Methods: Growth & Budget Forecasting

> **Target:** Forecast GWP by scheme and month for the budget year, with an uncertainty range and scenario levers
> **Index:** [00 - Analytics Programme Index](./00%20-%20Analytics%20Programme%20Index.md)

---

## 🏗️ Structure: Bottom-Up for Renewals, Top-Down for New Business

These two streams behave completely differently and must be modelled differently.

| | Renewal book | New business |
|---|---|---|
| Population known in advance? | **Yes** — you know exactly which policies expire when | No |
| Method | Aggregate policy-level probabilities | Time series / funnel decomposition |
| Main uncertainty | Retention rate, rate change | Volume, conversion, market conditions |
| Forecast error | Low (±3%) | High (±20%+) |

Most of your GWP is renewal, and it is the *forecastable* part. Get it right and the error bar on the total shrinks dramatically. Resist any pressure to model total GWP as one time series — it throws away the fact that you already know the expiry schedule.

---

## 1️⃣ Renewal Stream — Aggregating the Policy Model

For each future month $m$ and scheme $s$:

$$
\widehat{\text{RenewalGWP}}_{s,m} = \sum_{i \in \mathcal{E}_{s,m}} \hat{p}_i \cdot \text{GWP}_i^{\text{exp}} \cdot (1+\rho_{s}) \cdot (1+\epsilon_i)
$$

where $\mathcal{E}_{s,m}$ is the set of policies expiring in that month — **known from the policy system**, not forecast.

```python
scored["expected_gwp"] = (
    scored.p_renew * scored.expiring_gwp
    * (1 + scored.scheme.map(rate_plan))
    * (1 + scored.exposure_index)
)
walk = scored.groupby(["scheme", "expiry_month"]).expected_gwp.sum()
```

That is genuinely most of the work. The sophistication is all upstream in $\hat{p}_i$.

### Two things to get right

**Rate change must feed the retention model, not just multiply the output.** If the plan is +8% rate, re-score every policy with `rate_change_pct = 0.08` before summing. Applying rate as a post-hoc multiplier ignores the elasticity and will overstate the budget every time. This is the most common error in this whole exercise.

**Exposure change** ($\epsilon$) — sums insured, wageroll, turnover, vehicle counts drifting with inflation and client growth. Estimate from historic same-policy exposure movement, or index to an external series. Small, but it compounds and Finance will ask.

---

## 2️⃣ New Business — Funnel Decomposition

Don't forecast new business GWP directly. Decompose:

$$
N_{s,m} = \underbrace{Q_{s,m}}_{\text{quotes}} \times \underbrace{c_{s,m}}_{\text{conversion}} \times \underbrace{\bar{P}_{s,m}}_{\text{avg premium}}
$$

Each component is more forecastable and more explainable than the product:

- **Quote volume** — driven by broker activity, appointments, marketing, seasonality. Strong seasonality (January and renewal-season peaks). Model with STL + ETS, or a Poisson GLM with month dummies and broker-count as a regressor.
- **Conversion** — driven by price competitiveness and turnaround time. A logistic model on quote-level data if you have it; otherwise a seasonally-adjusted rate with a rate-position regressor.
- **Average premium** — trend with rate change and mix. Simple indexed trend usually suffices.

If quote-level data doesn't exist, escalate — capturing it is a higher-value project than refining the forecast. In the meantime, fall back to monthly new business GWP time series per scheme with a manual overlay for known pipeline (new binders, new broker appointments, planned scheme launches).

### Time series methods, in order of what to try

| Method | When | Package |
|---|---|---|
| Seasonal naive / STL + ETS | Default. 2–3 years of monthly data | `statsmodels`, `statsforecast` |
| SARIMAX with exogenous regressors | You have real drivers (broker count, rate index) | `statsmodels` |
| Poisson/negbin GLM on counts + separate severity | Count data, low volumes | `statsmodels` |
| Gradient boosting on lag features | 5+ years, many series | `lightgbm`, `mlforecast` |

> [!warning] With under ~36 monthly observations, anything more complex than ETS is fitting noise. Prophet and deep learning forecasters are not appropriate at MGA data volumes for a single scheme. Judgemental overlay from the underwriting pipeline will beat any of them for known new binders.

---

## 3️⃣ Hierarchical Reconciliation

Budgets are built at scheme level and rolled up. If you forecast each scheme independently, the parts won't sum to a sensibly forecast whole — and aggregate series are more stable and easier to forecast than their components.

Forecast at every level of the hierarchy (portfolio → product → scheme → broker), then **reconcile** so they're coherent:

```python
from hierarchicalforecast.core import HierarchicalReconciliation
from hierarchicalforecast.methods import MinTrace
rec = HierarchicalReconciliation(reconcilers=[MinTrace(method="mint_shrink")])
```

MinT (minimum trace) optimally combines the levels using the forecast error covariance — practically, it lets stable portfolio-level signal inform noisy scheme-level forecasts, and vice versa. Usually a meaningful accuracy gain, and it removes an entire category of "why doesn't this add up" meetings.

---

## 4️⃣ Uncertainty — Ship the Range

A point forecast will be treated as a commitment. Produce P10/P50/P90.

### Monte Carlo over the renewal book

```python
rng = np.random.default_rng(42)
sims = np.zeros(10_000)
for k in range(10_000):
    renewed = rng.random(len(p)) < p            # Bernoulli per policy
    gwp = (renewed * expiring_gwp * (1 + rate)).sum()
    gwp += rng.normal(nb_mu, nb_sigma)          # new business draw
    sims[k] = gwp
np.percentile(sims, [10, 50, 90])
```

Three caveats that matter:

1. **Independent Bernoullis understate the range badly.** Renewals are correlated — a broker leaves, a scheme gets repriced, the market hardens. Add a shared latent shock: draw a common factor $u \sim \mathcal{N}(0, \sigma_u)$ per broker/scheme per simulation and shift the log-odds of every policy in that group. Without this your P10–P90 band will be implausibly tight and nobody senior will believe it.
2. **Parameter uncertainty** — the retention model's own coefficients are estimated. Bootstrap the model, or use the Bayesian posterior, and resample coefficients per simulation.
3. **New business dominates the tail.** Its variance is much larger than the renewal book's; make sure the simulated distribution reflects that rather than a token normal draw.

### Scenario grid (what the business will actually ask for)

| Scenario | Rate change | Retention shock | New business |
|---|---|---|---|
| Plan | +6% | model | pipeline-based |
| Rate push | +12% | model (elasticity applies automatically) | −10% |
| Defend book | +2% | model | flat |
| Downside | +6% | −5pp uniform | −25% |
| Large broker loss | +6% | top broker set to 0 | flat |

The elasticity from [Note 02](./02%20-%20Methods%20-%20Retention%20Modelling.md#2%EF%B8%8F%E2%83%A3-splines--where-the-real-insight-lives) makes the first two scenarios *internally consistent* — retention falls automatically as rate rises. That consistency is the whole argument for building this rather than a spreadsheet.

---

## 5️⃣ Backtesting the Forecast

The naive benchmarks you must beat:

| Benchmark | Definition |
|---|---|
| Last year + flat growth | $\text{GWP}_{t-1} \times 1.05$ |
| Flat retention walk | Same walk, but $r$ = last year's overall retention rate, no model |
| Finance's prior-year budget | Their actual last budget vs actual outcome |

That third one matters politically. If the model's backtested error is materially lower than the human budget's historical error, the case makes itself. Compute it — the numbers are sitting in old budget files.

**Metrics:** MAPE and bias (mean error, signed) at portfolio and scheme level, plus month-level for phasing. Bias matters more than MAPE for budgeting — a forecast that's consistently 3% high is a worse problem than one that's noisily ±5%.

**Protocol:** rolling origin. Refit everything as at 1 Oct of year $t-1$ using only data available then, forecast year $t$, compare to actual. Repeat for every year you have. Automate this — you'll rerun it every time someone challenges a number.

---

## 📊 What Goes to Qlik

Three consumers, three apps. Qlik reads scored tables from SQL MI; **no modelling in Qlik**.

| App | Consumer | Content |
|---|---|---|
| **Budget bridge** | CFO, exec | Waterfall of the renewal walk, scheme drill-down, scenario selector, P10/P50/P90 band, plan-vs-forecast-vs-actual |
| **Retention monitor** | Underwriting, ops | Actual vs predicted retention by scheme/broker/month, rate-change vs retention scatter, elasticity curves, A/E trend |
| **Renewal risk worklist** | Account handlers | Policies expiring in next 90 days, sorted by value at risk, with top 3 drivers and a contacted flag |

The worklist is the one that gets used daily and secures phase 2 funding. Build it in phase 3, not "later".

Tables to expose: `Analytics.RenewalScore` (policy, p_renew, drivers, value_at_risk), `Analytics.ForecastWalk` (scheme, month, scenario, walk_line, value), `Analytics.ForecastSimulation` (scheme, month, percentile, value).

---

*[Index](./00%20-%20Analytics%20Programme%20Index.md) | Prev: [02 - Methods - Retention Modelling](./02%20-%20Methods%20-%20Retention%20Modelling.md) | Next: [04 - Delivery Validation and Governance](./04%20-%20Delivery%20Validation%20and%20Governance.md)*
