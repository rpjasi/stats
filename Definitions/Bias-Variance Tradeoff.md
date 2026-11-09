# Bias-Variance Tradeoff

> **Category:** Model Assessment  
> **First appears:** [Ch02 - Statistical Learning](../Chapters/Ch02 - Statistical Learning.md)  
> **Back to glossary:** [00 - Glossary](00 - Glossary.md)

---

## Plain English Definition

Every model makes two types of errors: bias (wrong assumptions baked in — the model is too simple to capture reality) and variance (too sensitive to the specific training data — it memorises noise). Making the model more complex reduces bias but increases variance, and vice versa. The goal is to find the sweet spot that minimises total test error.

---

## Formal Definition

For a model $\hat{f}$ trained on data and evaluated at a point $x_0$:

$$
\mathbb{E}\left[(y_0 - \hat{f}(x_0))^2\right] = \underbrace{\text{Var}(\hat{f}(x_0))}_{\text{variance}} + \underbrace{\left[\text{Bias}(\hat{f}(x_0))\right]^2}_{\text{bias}^2} + \underbrace{\text{Var}(\epsilon)}_{\text{irreducible error}}
$$

| Symbol | Meaning |
|---|---|
| $y_0$ | true response at test point $x_0$ |
| $\hat{f}(x_0)$ | model's prediction at $x_0$ |
| $\text{Var}(\hat{f})$ | how much predictions vary across different training sets |
| $\text{Bias}(\hat{f})$ | systematic error from wrong modelling assumptions |
| $\text{Var}(\epsilon)$ | irreducible noise — cannot be eliminated by any model |

---

## Intuitive Analogy

Think of hitting a target with arrows. **High bias, low variance** = all your arrows cluster tightly together but in the wrong place (consistently wrong). **Low bias, high variance** = arrows are centred on the bullseye on average, but scattered all over the target (right on average, but unpredictable). You want tight *and* centred — that's the tradeoff.

---

## When to Use It

- ✅ Use this framework when deciding model complexity (e.g. polynomial degree, tree depth, number of neighbours in KNN)
- ✅ Use when diagnosing whether your model is overfitting or underfitting
- ✅ Use when comparing regularisation strengths ($\lambda$ in Ridge/Lasso)

## When It Breaks Down

- ❌ The clean decomposition assumes squared error loss — doesn't directly apply to classification
- ❌ In practice you can't directly measure bias and variance separately without repeated sampling

---

## Related Terms

- [Overfitting vs Underfitting](./Overfitting vs Underfitting.md) — the practical manifestation of this tradeoff
- [Mean Squared Error (MSE)](./Mean Squared Error (MSE).md) — the quantity being decomposed
- [Cross-Validation](./Cross-Validation.md) — the tool used to estimate test error and find the sweet spot
- [Irreducible Error](./Irreducible Error.md) — the $\text{Var}(\epsilon)$ floor that no model can beat
- [Ridge Regression](./Ridge Regression.md) — adds regularisation to manage variance
- [Lasso](./Lasso.md) — same idea, with sparsity

---

## Seen In

- [Ch02 - Statistical Learning](../Chapters/Ch02 - Statistical Learning.md) — introduced and derived
- [Ch05 - Resampling Methods](../Chapters/Ch05 - Resampling Methods.md) — cross-validation used to navigate the tradeoff
- [Ch06 - Linear Model Selection](../Chapters/Ch06 - Linear Model Selection.md) — regularisation as a tool to manage variance

---

*[00 - Glossary](00 - Glossary.md) | [00 - Index](../00 - Index.md)*
