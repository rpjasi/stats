# Ch2 - Statistical Learning

## Summary

Chapter 2 introduces the statistical learning framework by writing the response as
$Y = f(X) + \epsilon$ and asking how we can estimate the unknown function $f$ from data.
It separates two goals that often compete with each other: prediction and inference.
The chapter also formalizes training versus test error, explains why test error is the
real quantity of interest, and derives the bias-variance decomposition for expected
test MSE at a point $x_0$. It then contrasts parametric and non-parametric methods and
uses K-nearest neighbours (KNN) as the main example of a flexible non-parametric
classifier. This chapter is the conceptual base for the rest of ISLP because later
methods are all different ways of controlling flexibility, bias, variance, and test
error. (§2.1-2.2, pp. 15-46)

---

## Key Concepts

- **[[Definitions/Statistical Learning]]** — Statistical learning studies how to estimate an unknown function $f$ in the model $Y = f(X) + \epsilon$. The estimate $\hat{f}$ can be used either for accurate prediction or for understanding how predictors relate to the response.
- **[[Definitions/Bias-Variance Tradeoff|bias-variance trade-off]]** — Expected test MSE at a point $x_0$ decomposes into variance, squared bias, and irreducible error. More flexibility usually lowers bias but raises variance. (§2.2.2, p. 34)
- **[[Definitions/Supervised Learning]]** — In supervised learning, each observation includes both predictors $X$ and a response $Y$, so the goal is to learn a rule relating them. (§2.1.4, p. 26)
- **[[Definitions/Unsupervised Learning]]** — In unsupervised learning, no response $Y$ is observed, so the goal is to find structure in the predictors alone. (§2.1.4, p. 26)
- **[[Definitions/Regression]]** — Regression problems have a quantitative response $Y$. Test MSE is the default loss function discussed in this chapter. (§2.1.5, p. 28)
- **[[Definitions/Classification]]** — Classification problems have a qualitative response $Y$. KNN classification is the chapter's main non-parametric example. (§2.1.5, p. 28)
- **[[Definitions/Clustering]]** — Clustering is used as the canonical unsupervised example: no labels, only structure discovery. (§2.1.4, p. 27)
- **Parametric methods** — These methods assume a specific functional form for $f$ and estimate a finite number of parameters. They are easier to interpret but can be misspecified. (§2.1.2, pp. 21-23)
- **Non-parametric methods** — These methods avoid specifying a fixed form for $f$ and instead let the data determine its shape. They are flexible but usually require more data. (§2.1.2, pp. 23-24)
- **Prediction vs inference** — Prediction cares about how well $\hat{f}(X)$ forecasts unseen outcomes; inference cares about understanding the effect of predictors on $Y$. (§2.1.3, pp. 24-26)
- **Training error vs test error** — Training error is measured on the data used to fit the model, while test error is measured on unseen data. Only test error tells us about generalization. (§2.2.1, pp. 29-31)
- **K-nearest neighbours (KNN)** — KNN estimates local structure by averaging or voting over the $K$ closest training points to a test point $x_0$. Small $K$ gives high flexibility; large $K$ gives smoother, less variable fits. (§2.2.3, pp. 38-42)

---

## Core Formulas

### The Statistical Learning Model

$$
Y = f(X) + \epsilon
$$

| Symbol | Meaning |
|---|---|
| $Y$ | Response variable |
| $X$ | Predictor or feature vector |
| $f$ | Fixed but unknown systematic relationship between $X$ and $Y$ |
| $\epsilon$ | Random error term with mean zero, independent of $X$ |

_Equation (2.1), p. 16._

---

### Training MSE

$$
\mathrm{MSE} = \frac{1}{n}\sum_{i=1}^{n}\left(y_i - \hat{f}(x_i)\right)^2
$$

| Symbol | Meaning |
|---|---|
| $n$ | Number of training observations |
| $y_i$ | Observed response for observation $i$ |
| $\hat{f}(x_i)$ | Prediction at predictor value $x_i$ |

_Section 2.2.1, p. 29._

---

### Expected Test Error at a Point

$$
\mathbb{E}\left(y_0 - \hat{f}(x_0)\right)^2
$$

This is the pointwise expected test MSE for a new observation $(x_0, y_0)$.

_Section 2.2.2, p. 34._

---

### Bias-Variance Decomposition

$$ 
\mathbb{E}\left(y_0 - \hat{f}(x_0)\right)^2 = \operatorname{Var}\!\left(\hat{f}(x_0)\right) + \left[\operatorname{Bias}\!\left(\hat{f}(x_0)\right)\right]^2 + \operatorname{Var}(\epsilon)
 $$

| Symbol | Meaning |
|---|---|
| $x_0$ | A specific test point |
| $\operatorname{Var}(\hat{f}(x_0))$ | Variability of the fitted prediction across training samples |
| $\operatorname{Bias}(\hat{f}(x_0))$ | Systematic error from approximating $f$ with the chosen method |
| $\operatorname{Var}(\epsilon)$ | Irreducible noise in the data-generating process |

_Equation (2.7), p. 34._

---

### KNN Estimated Class Probability

$$
\Pr(Y=j \mid X=x_0) = \frac{1}{K}\sum_{i \in \mathcal{N}_0} I(y_i = j)
$$

| Symbol | Meaning |
|---|---|
| $K$ | Number of nearest neighbours |
| $\mathcal{N}_0$ | Set of the $K$ observations closest to $x_0$ |
| $I(y_i = j)$ | Indicator equal to 1 if observation $i$ belongs to class $j$, else 0 |
| $x_0$ | Test point being classified |

_Equation (2.12), p. 39._

---

### Test MSE

$$
\mathrm{MSE}_{\mathrm{test}} = \frac{1}{n_0}\sum_{i=1}^{n_0}\left(y_{0i} - \hat{f}(x_{0i})\right)^2
$$

| Symbol | Meaning |
|---|---|
| $n_0$ | Number of test observations |
| $y_{0i}$ | True response for test observation $i$ |
| $\hat{f}(x_{0i})$ | Prediction for test observation $i$ |

_Section 2.2.1, p. 29._

---

## Intuition

Think of Chapter 2 as the chapter that explains why fitting the training data is not
the same thing as learning the truth. There is a real signal, represented by $f(X)$,
but every dataset also contains noise $\epsilon$. If your model is too simple, it
misses real structure and has high bias. If it is too flexible, it starts memorizing
noise and has high variance. The practical problem is therefore not "fit as closely as
possible," but "choose a level of flexibility that minimizes test error."

---

## Statistical Assumptions

- **$\mathbb{E}(\epsilon) = 0$** — The noise has mean zero, so the systematic part of the model is captured by $f(X)$ rather than by the error term.
- **$\epsilon$ is independent of $X$** — If the error term depends on the predictors, then part of the signal has been pushed into the noise term and inference becomes distorted.
- **Observations are i.i.d.** — Much of the theory in the chapter assumes independently and identically distributed observations. Time dependence or clustering breaks this setup.
- **$f$ is fixed but unknown** — The framework assumes there is a stable underlying relationship to estimate. If the data-generating process shifts, test performance can degrade.
- **For KNN, distance must be meaningful** — KNN depends on the geometry of predictor space, so predictors should be on comparable scales and not dominated by irrelevant dimensions.

---

## Decision Guide

| Situation | Use this chapter's framework | Consider instead |
|---|---|---|
| Quantitative response, interpretability matters | Parametric methods such as linear regression | More flexible non-parametric methods if sample size is large |
| Qualitative response, local structure matters | KNN classification | Logistic regression or tree-based methods if you need a different decision boundary or better scaling |
| Very small $n$, many predictors $p$ | Simpler parametric models or regularization | Avoid vanilla KNN because neighbourhoods become unstable |
| Large sample, complex decision boundary | Flexible non-parametric methods | Simpler parametric methods if interpretability dominates |
| Goal is explanation rather than forecasting | Inference-oriented parametric models | Purely predictive black-box methods |

---

## Python Implementation

```python
import numpy as np
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.preprocessing import StandardScaler

# Simulate a binary classification problem.
np.random.seed(42)
X = np.random.randn(200, 2)
y = (X[:, 0] + X[:, 1] > 0).astype(int)

# Hold out test data because Chapter 2 focuses on test error, not training error.
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42
)

# KNN depends on distances, so scale features first.
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

for K in [1, 5, 15, 50]:
    knn = KNeighborsClassifier(n_neighbors=K, metric="euclidean")
    knn.fit(X_train_s, y_train)

    train_acc = accuracy_score(y_train, knn.predict(X_train_s))
    test_acc = accuracy_score(y_test, knn.predict(X_test_s))
    print(f"K={K:3d} | train={train_acc:.3f} | test={test_acc:.3f}")
```

---

## Common Pitfalls

- ⚠️ **Optimizing training error instead of test error** — Chapter 2 is explicit that training error is usually too optimistic.
- ⚠️ **Ignoring irreducible error** — Even the best method cannot eliminate $\operatorname{Var}(\epsilon)$.
- ⚠️ **Treating small $K$ as automatically better** — In KNN, very small $K$ lowers bias but can explode variance.
- ⚠️ **Using KNN without scaling predictors** — Distances become dominated by large-scale features.
- ⚠️ **Using a highly flexible model for an inference question** — Prediction quality and interpretability are different goals.

---

## Connections to Other Chapters

- **Builds on:** [[Chapters/Ch01 - Introduction]] — Chapter 1 defines the problem types; Chapter 2 formalizes the framework.
- **Leads to:** [[Chapters/Ch03 - Linear Regression]] — linear regression is the first concrete parametric model for estimating $f$.
- **Leads to:** [[Chapters/Ch05 - Resampling Methods]] — resampling is how we estimate test error in practice.
- **Leads to:** [[Chapters/Ch06 - Linear Model Selection]] — regularization is a practical way to manage the bias-variance trade-off.
- **Underpins:** [[Chapters/Ch08 - Tree-Based Methods]] and [[Chapters/Ch09 - Support Vector Machines]] — later supervised methods can all be understood through flexibility, bias, variance, and test error.

---

## Review Questions

**Q1:** What are the three terms in the bias-variance decomposition, and which one cannot be reduced by improving the model?

> [!note]- Answer
> The decomposition consists of variance, squared bias, and irreducible error:
> $\operatorname{Var}(\hat{f}(x_0))$, $[\operatorname{Bias}(\hat{f}(x_0))]^2$, and
> $\operatorname{Var}(\epsilon)$. The irreducible error term cannot be reduced by
> changing the model.

**Q2:** Why can a model with extremely low training error still have poor test performance?

> [!note]- Answer
> Because low training error may reflect overfitting rather than learning the true
> signal. A highly flexible model can adapt to random noise in the training sample,
> increasing variance and therefore increasing test error.

**Q3:** What happens in KNN as $K$ becomes very large?

> [!note]- Answer
> The fit becomes smoother and less variable, but bias increases. In the limit, KNN
> approaches a very inflexible rule that can underfit the data.

---

## My Notes & Observations

---

_[[00 - Index]] | [[Labs/Lab02 - Statistical Learning]] | Next: [[Chapters/Ch03 - Linear Regression]]_
