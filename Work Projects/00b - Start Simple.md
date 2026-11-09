# 00b — Start Simple: The Escalation Ladder

> **Read this before the other notes.** Notes 01–04 describe the destination. This one describes the shortest route to something useful, and when to climb.
> **Index:** [00 - Analytics Programme Index](./00%20-%20Analytics%20Programme%20Index.md)

---

## 🪜 The Ladder

| Level | Effort | What you build | What it gives you | What it can't do |
|---|---|---|---|---|
| **0 — Arithmetic** | 2 days | Expiring GWP × last year's retention × planned rate | A budget number, and a baseline everything else must beat | Nothing segment-level, no scenarios |
| **1 — Segmented rates** | 2–3 weeks | Same walk, retention rates split by scheme × rate band × premium band. SQL + Qlik only | A defensible budget, visible retention drivers, a crude elasticity table | No policy-level scores, no worklist, no uncertainty range |
| **2 — Logistic model** | 6–8 weeks | Policy-level renewal probabilities in Python, scored back to SQL MI | At-risk worklist, real elasticity curve, consistent scenarios | Interaction effects, proper uncertainty, automation |
| **3 — Full programme** | 3–6 months | Splines, pooling, GBM challenger, Monte Carlo, reconciliation, scheduled Azure pipeline | Notes [01](./01%20-%20Phase%201%20Approach.md)–[04](./04%20-%20Delivery%20Validation%20and%20Governance.md) | — |

**Level 1 is the important rung.** It delivers most of the business value, forces every hard definitional question into the open, and proves the data exists — all before you write a line of modelling code. It is also the level most teams skip, which is why their level 2 fails.

---

## Level 0 — The Baseline (do this in week 1 regardless)

$$
\text{Forecast} = \text{Expiring GWP} \times r_{\text{last year}} \times (1 + \rho) + \text{New business}_{\text{last year}}
$$

Not a deliverable. It's the number every later model must beat, and you need it on record before you start so nobody relitigates the benchmark later.

---

## Level 1 — Segmented Rates (the recommended starting point)

**Build:** one SQL table of renewal outcomes for the last 2–3 years, one row per renewal opportunity. Then a pivot: retention rate by scheme × rate-change band × premium band. Apply those rates to next year's known expiry schedule. Present as the renewal walk.

**Stack:** SQL MI + Qlik. No Python, no Azure, no models.

**You get:**

- A budget built bottom-up from the actual expiry schedule rather than a growth assumption
- Retention visibly varying by rate change — usually the first time anyone has seen this quantified
- A scheme-level view of where the book is leaking
- The `RenewalOpportunity` table, which is 80% of the work for level 2

**The three questions this forces you to answer** — and which will consume most of the effort:

1. What exactly counts as a renewal? (invited vs expiring, count vs premium, what's excluded)
2. Can you reconstruct history point-in-time, or has the policy table been overwritten?
3. Does your retention number tie to Finance's?

**Stop here if:** the retention rate barely varies within scheme, the book is small (<5k renewals/year), or nobody has capacity to act on policy-level scores. A model that ranks policies is worthless if no one calls the at-risk ones.

---

## Level 2 — Logistic Model

**Build:** logistic regression on the level-1 table. Score every upcoming renewal. Write probabilities back to SQL MI. Qlik reads them.

**Adds over level 1:**

- Policy-level risk → an at-risk worklist sorted by `(1 − p) × expiring GWP`
- A real elasticity curve, so "+10% rate" scenarios are internally consistent
- Effects estimated jointly rather than sliced into ever-thinner cells

**Climb to level 2 when:** level 1 cells are getting too thin to be stable, *or* someone asks "which renewals should I call first?", *or* underwriting wants to price a rate scenario.

Keep it deliberately plain: one model, all schemes, scheme as a feature. Resist per-scheme models until you have evidence you need them.

---

## Level 3 — The Full Programme

Everything in notes [01](./01%20-%20Phase%201%20Approach.md)–[04](./04%20-%20Delivery%20Validation%20and%20Governance.md). Climb one rung at a time, and only when a specific question demands it:

| Add this | Only when |
|---|---|
| Splines on rate change | The linear elasticity is visibly wrong at the extremes |
| Hierarchical pooling | Broker-level estimates are needed and thin brokers are unstable |
| GBM challenger | You need to prove the simple model isn't missing much |
| Monte Carlo range | Someone treats the point forecast as a commitment |
| Automated Azure pipeline | Manual reruns become the bottleneck |
| Survival / hazard | Mid-term cancellation becomes material |

---

## 🧭 Two Rules

**Ship something every 4–6 weeks.** A visible level-1 output that Finance uses beats a level-3 model still in a branch. Analytics programmes get cancelled for lack of visible output far more often than for lack of accuracy.

**Every rung must beat the one below it, measured.** If the logistic model doesn't beat segmented rates on a backtested year, don't ship it — and that's a real possibility on a small book. Knowing that early is worth more than the model.

---

*[Index](./00%20-%20Analytics%20Programme%20Index.md) | Next: [01 - Phase 1 Approach](./01%20-%20Phase%201%20Approach.md)*
