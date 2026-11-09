# 02 — Methods: Retention Modelling

> **Target:** $P(\text{renew} = 1 \mid \mathbf{x}_{\text{snapshot}})$ at policy level
> **Index:** [00 - Analytics Programme Index](./00%20-%20Analytics%20Programme%20Index.md)

---

## 🪜 The Model Ladder

Climb it in order. Stop when the next rung stops paying for itself.

| Rung | Method | Gains you | Costs you | ISLP |
|---|---|---|---|---|
| 0 | Flat prior-year retention rate | Benchmark to beat | Nothing | — |
| 1 | Logistic regression / binomial GLM | Interpretable coefficients, odds ratios | Assumes linear log-odds | [Ch04](../Chapters/Ch04%20-%20Classification.md) |
| 2 | GLM + splines on rate change & tenure | The elasticity **curve**, not a slope | Some interpretability | [Ch07](../Chapters/Ch07%20-%20Moving%20Beyond%20Linearity.md) |
| 3 | Hierarchical GLM (broker/scheme random effects) | Sane estimates for thin brokers | Fitting complexity | — |
| 4 | Gradient boosting (LightGBM/XGBoost) | Interactions, accuracy ceiling | Explainability, governance | [Ch08](../Chapters/Ch08%20-%20Tree-Based%20Methods.md) |
| 5 | Discrete-time hazard / survival | *When* they lapse, handles censoring | Reshaping data to person-period | [Ch11](../Chapters/Ch11%20-%20Survival%20Analysis.md) |

For a first budget cycle, rungs 1–3 are almost certainly sufficient. Rung 4 is a challenger to quantify how much structure you're missing.

---

## 1️⃣ Logistic Baseline

$$
\log\frac{p_i}{1-p_i} = \beta_0 + \beta_1 \Delta\text{rate}_i + \beta_2 \text{tenure}_i + \boldsymbol{\beta}_3^\top \mathbf{z}_i
$$

```python
import statsmodels.formula.api as smf

m = smf.glm(
    "renewed_flag ~ rate_change_pct + tenure_years + C(payment_method) "
    "+ np.log(expiring_gwp) + claims_count_3yr + C(product)",
    data=train,
    family=sm.families.Binomial(),
    freq_weights=None,          # weight by expiring_gwp for a premium-weighted fit
).fit()
```

**Read the coefficients as business quantities.** $\beta_1$ on `rate_change_pct` is your headline elasticity: a coefficient of $-0.045$ means each +1pp of rate multiplies the renewal odds by $e^{-0.045} \approx 0.956$. Convert to a marginal effect at the mean before quoting it to anyone — odds ratios get misread as probabilities constantly.

**Weighting.** Fit unweighted for a policy-count model; fit weighted by `expiring_gwp` when the output feeds the premium-weighted walk. Do both — the divergence between them is itself a finding (are you losing lots of small policies or a few big ones?).

---

## 2️⃣ Splines — Where the Real Insight Lives

The rate change → retention relationship is not linear in log-odds. It's typically flat to about +5%, then falls away sharply, with a further cliff at broker-shopping thresholds. A linear term averages that S-curve into a single misleading slope, and then your +10% scenario is badly wrong.

```python
from patsy import dmatrix   # natural cubic spline, knots at business-meaningful points
"renewed_flag ~ cr(rate_change_pct, df=5) + cr(tenure_years, df=4) + ..."
```

Place knots deliberately — at 0%, +5%, +10%, +20% — rather than at quantiles. The knots are the story.

**Output the curve, not the model.** For each scheme, produce a table of retention at rate changes from −5% to +25%. That single artefact will get more use than everything else you build, because it's what underwriting needs to set the plan.

---

## 3️⃣ Hierarchical / Partial Pooling

MGA books are broker-concentrated with a long tail. A broker with 9 renewals and 9 lapses will get a coefficient of $-\infty$ under fixed effects, and dropping them loses real signal. Partial pooling shrinks thin brokers toward the scheme mean.

$$
\log\frac{p_{ij}}{1-p_{ij}} = \alpha_{j[i]} + \mathbf{x}_i^\top\boldsymbol{\beta},
\qquad
\alpha_j \sim \mathcal{N}(\mu_{\text{scheme}}, \sigma^2)
$$

Options, cheapest first:

- **Regularised fixed effects** — one-hot brokers + L2 penalty (`sklearn.linear_model.LogisticRegression(penalty="l2")`). Crude pooling, 20 minutes of work, usually fine. See [Ch06](../Chapters/Ch06%20-%20Linear%20Model%20Selection.md).
- **Target/GLMM encoding** — empirical-Bayes shrunk broker retention rate as a feature. Watch for leakage: compute it out-of-fold.
- **Proper GLMM** — `statsmodels` `BinomialBayesMixedGLM`, or `pymc` / `numpyro` for a full Bayesian fit with credible intervals on broker effects.

The Bayesian route is worth it if the business wants to rank brokers by retention quality — you get intervals, so you can say "we can't distinguish these two brokers" instead of producing a spurious league table.

---

## 4️⃣ Gradient Boosting Challenger

```python
import lightgbm as lgb
model = lgb.LGBMClassifier(
    objective="binary", n_estimators=2000, learning_rate=0.03,
    num_leaves=31, min_child_samples=100,     # keep leaves fat; MGA books are small
    subsample=0.8, colsample_bytree=0.8,
)
```

Use it to answer one question: **how much accuracy is the GLM leaving on the table?** If the gap is small, you have a defensible reason to ship the interpretable model. If the gap is large, use SHAP to find the interaction the GLM is missing, then *add that interaction to the GLM*.

Guardrails: MGA renewal datasets are often 20k–200k rows, which is small for boosting. Set `min_child_samples` high, use early stopping on a temporally held-out set, and never tune on the same period you evaluate on.

---

## 5️⃣ Survival / Hazard Framing

Two uses:

1. **Mid-term cancellation** — genuinely a time-to-event problem with censoring. A Cox model or discrete-time logistic hazard on policy-months gives you $C_{t+1}$ in the walk, and a cancellation risk score.
2. **Censored renewals** at forecast time — policies expiring inside your observation window whose outcome isn't known yet.

Discrete-time hazard is the pragmatic version: expand to one row per policy-month, fit a logistic model with month-since-inception as a spline, and you have a hazard curve without leaving GLM tooling. See [Ch11](../Chapters/Ch11%20-%20Survival%20Analysis.md).

---

## 🧰 Feature Set

Ranked by typical signal in commercial/specialty MGA books:

| Tier | Features | Note |
|---|---|---|
| **Strong** | `rate_change_pct`, `price_to_technical_ratio`, `payment_method` (DD vs annual), `tenure_years`, `auto_renew_flag` | Rate change and payment method usually dominate |
| **Moderate** | `broker_id` effects, `scheme_id`, `expiring_gwp` band, `claims_count_3yr`, `prior_renewals`, `mta_count` | Claims cut both ways: claimants who were paid often *stay* |
| **Contextual** | `region`, `sic_code`/trade, `channel`, `capacity_provider`, seasonality of expiry month | |
| **Operational (high value, often missing)** | days between renewal invite and expiry, whether a renewal call was made, quote turnaround time, number of contacts | If you can get these, they're often the biggest actionable lever — and unlike rate, they're free to change |
| **External** | market rate index, competitor pricing proxy, Lloyd's/market cycle indicators | Hard to source; skip in phase 1 |

### Engineering notes

- Rate change: use **applied** rate change, not headline scheme rate. They differ once underwriter discretion is involved — and the gap is a feature.
- Premium: log-transform. Retention behaviour is roughly linear in log premium, not premium.
- Claims: separate "had a claim" from "claim amount" — the relationship isn't monotonic.
- Broker: never raw ID into a GLM without pooling or regularisation.
- Missingness in MGA data is usually **informative** (blank `sum_insured` means a different product path). Add explicit missing-indicator columns rather than imputing silently.

---

## ⚠️ Leakage — The Main Way This Goes Wrong

Every one of these has bitten a real project:

| Leak | How it happens | Fix |
|---|---|---|
| Renewal premium as a feature | It only exists if they renewed | Snapshot discipline |
| "Renewal quote issued" flag | Populated during the decision | Only use if timestamped before snapshot |
| Current broker on the policy record | Updated when business moves | Use as-at-snapshot broker |
| Claims records with movement dates after snapshot | Claims reserves get restated | Filter claim rows by `transaction_date <= snapshot_date` |
| Policy status field | Overwritten to "Lapsed" | Never use status; derive target from the existence of a subsequent term |
| Random train/test split | Leaks future into past | Time-based split only |

> [!warning] If your first model gets 0.95 AUC on renewal prediction, you have leakage. Realistic AUC for commercial renewal is roughly **0.70–0.82**. Treat a great score as a bug report.

---

## ✅ Evaluation — Calibration Over Discrimination

You are summing predicted probabilities to produce a GWP number. That makes **calibration** the primary metric.

| Metric | What it tells you | Target |
|---|---|---|
| **Calibration curve / reliability plot** | Do 80% predictions renew 80% of the time | Within ±3pp across deciles |
| **Brier score** | Combined calibration + sharpness | Beat the flat-rate baseline |
| **Portfolio A vs E** | $\sum p_i$ vs actual renewals, and premium-weighted | Within ±2% |
| **A vs E by segment** | Same, split by scheme / broker / rate band / premium size | No segment out by >5% |
| AUC / Gini | Ranking quality — matters for the worklist, not the budget | 0.70+ |
| Lift in top decile | Is the at-risk worklist worth an account handler's time | 2×+ |

**Segment-level A/E is the acceptance test.** A model that is right in aggregate but systematically wrong on your two largest schemes is worse than useless, because the budget is built by scheme.

Recalibrate if needed — Platt scaling or isotonic regression on a held-out period is legitimate and cheap.

### Validation protocol

Time-based, always:

```
Train:  2022-01 → 2024-12 renewals
Val:    2025-01 → 2025-06   (tuning, early stopping, calibration)
Test:   2025-07 → 2026-06   (touched once, at the end)
```

Then **rolling-origin backtest** across as many origins as your history allows: refit at each origin, forecast forward, compare. This is what proves the approach works, not a single holdout. Note that ordinary k-fold CV ([Ch05](../Chapters/Ch05%20-%20Resampling%20Methods.md)) is the wrong tool here despite being the default reflex.

---

## 🎬 From Score to Action

The model output must land in two places:

1. **Aggregated** → the renewal walk → [Note 03](./03%20-%20Methods%20-%20Growth%20and%20Budget%20Forecasting.md)
2. **Per policy** → an at-risk worklist for account handlers, 60–90 days ahead of expiry, with the top 3 drivers per policy (SHAP values, or GLM term contributions) and expiring GWP so they triage by value at risk, not just probability.

Value at risk $= (1 - p_i) \times \text{expiring GWP}_i$. Sort by that, not by $p_i$. A 40% lapse chance on £120k beats a 90% lapse chance on £900.

---

*[Index](./00%20-%20Analytics%20Programme%20Index.md) | Prev: [01 - Phase 1 Approach](./01%20-%20Phase%201%20Approach.md) | Next: [03 - Methods - Growth and Budget Forecasting](./03%20-%20Methods%20-%20Growth%20and%20Budget%20Forecasting.md)*
