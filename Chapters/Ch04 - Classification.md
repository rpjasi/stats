---
# Ch4 - Classification

> **Textbook pages:** pp. 129-199
> **Date studied:**
> **Confidence:** 🟡 Partial
> **Linked lab:** [[Labs/Lab04 - Classification]]
> **Back to index:** [00 - Index](../00%20-%20Index.md)

---

## Summary

Chapter 4 addresses classification: predicting a qualitative response such as whether a customer will default or which diagnosis applies. It explains why ordinary linear regression is generally unsuitable for categorical outcomes and introduces logistic regression, which models class probabilities through the log-odds. It then develops generative classifiers - linear discriminant analysis (LDA), quadratic discriminant analysis (QDA), and naive Bayes - by modelling how predictors are distributed within each class and applying Bayes' theorem. The chapter compares these methods with K-nearest neighbours (KNN), showing that their relative performance depends on whether the true decision boundary is linear or non-linear and on the bias-variance trade-off. It also shows how thresholds, confusion matrices, sensitivity, specificity, ROC curves, and AUC assess a classifier beyond raw accuracy. Finally, it places logistic and Poisson regression inside the generalized linear model framework, linking classification to the broader regression ideas from Chapter 3.

## Key Concepts

- **[Classification](../Definitions/Classification.md)** - Predicts a qualitative response by estimating class probabilities or assigning an observation to one of a finite number of classes (Section 4.1, pp. 129-132).
- **[Logistic Regression](../Definitions/Logistic%20Regression.md)** - Models the log-odds of a binary outcome as a linear function of the predictors, ensuring fitted probabilities remain between 0 and 1 (Section 4.3, pp. 133-140).
- **[Logistic Function](../Definitions/Logistic%20Function.md)** - Converts any real-valued linear score into a probability in the interval $(0,1)$ (Eq. 4.2, p. 133).
- **[Odds and Log-Odds](../Definitions/Odds%20and%20Log-Odds.md)** - Odds compare the probability of an event with its non-occurrence; taking their logarithm produces the logit, which logistic regression models linearly (Eqs. 4.3-4.4, p. 134).
- **[Maximum Likelihood](../Definitions/Maximum%20Likelihood.md)** - Estimates model coefficients by choosing values that make the observed class labels most probable under the model (Eq. 4.5, p. 135).
- **[Multinomial Logistic Regression](../Definitions/Multinomial%20Logistic%20Regression.md)** - Extends logistic regression from two classes to $K$ classes using class-specific linear scores and softmax probabilities (Eqs. 4.10-4.14, pp. 139-140).
- **[Bayes Classifier](../Definitions/Bayes%20Classifier.md)** - Assigns an observation to the class with the largest true posterior probability and achieves the lowest theoretically possible error rate (Section 4.4, pp. 146-147).
- **[Bayes' Theorem](../Definitions/Bayes%20Theorem.md)** - Combines a class prior $\pi_k$ with the class-conditional density $f_k(x)$ to obtain the posterior probability that an observation belongs to class $k$ (Eq. 4.15, p. 147).
- **[Linear Discriminant Analysis](../Definitions/LDA.md)** - Estimates class-specific means and a covariance matrix shared by all classes; this shared-covariance assumption produces linear decision boundaries (Section 4.4.1, pp. 147-154).
- **[Confusion Matrix](../Definitions/Confusion%20Matrix.md)** - Cross-tabulates predicted and actual classes to expose which types of errors a classifier makes (pp. 145, 154-155).
- **[Sensitivity and Specificity](../Definitions/Sensitivity%20and%20Specificity.md)** - Sensitivity is the true-positive rate, while specificity is the true-negative rate; changing the classification threshold trades one against the other (Section 4.4.2, pp. 156-159).
- **[ROC Curve and AUC](../Definitions/ROC%20Curve%20and%20AUC.md)** - The ROC curve plots sensitivity against $1-\text{specificity}$ over all thresholds; AUC summarizes the classifier's threshold-independent discrimination (pp. 158-159).
- **[Quadratic Discriminant Analysis](../Definitions/QDA.md)** - Gives each class its own covariance matrix, allowing quadratic decision boundaries at the cost of estimating more parameters (Section 4.4.3, pp. 159-161).
- **[Naive Bayes](../Definitions/Naive%20Bayes.md)** - Estimates each predictor's distribution separately within each class by assuming conditional independence; the strong assumption can greatly reduce variance (Section 4.4.4, pp. 161-164).
- **[K-Nearest Neighbours](../Definitions/K-Nearest%20Neighbours.md)** - A non-parametric classifier that estimates a class probability from the labels of the $K$ closest training observations; it can learn highly flexible boundaries but may have high variance (Section 4.5, pp. 164-166).
- **[Decision Boundary](../Definitions/Decision%20Boundary.md)** - The set of predictor values at which the predicted class changes; logistic regression and LDA create linear boundaries, while QDA and flexible KNN can create non-linear ones.
- **[Generalized Linear Model](../Definitions/Generalized%20Linear%20Model.md)** - Relates a response distribution's mean to a linear predictor through a link function, unifying linear, logistic, and Poisson regression (Section 4.6, pp. 167-172).
- **[Poisson Regression](../Definitions/Poisson%20Regression.md)** - Models count data using a Poisson response distribution and a log link so that the predicted mean is always positive (Section 4.6.1, pp. 168-171).

## Core Formulas

### Binary Logistic Probability

$$
p(X)=\Pr(Y=1\mid X)=\frac{e^{\beta_0+\beta_1X}}{1+e^{\beta_0+\beta_1X}}
$$

| Symbol | Meaning |
|---|---|
| $p(X)$ | Conditional probability that $Y=1$ given predictors $X$ |
| $Y$ | Binary response, coded 0 or 1 |
| $X$ | Predictor value |
| $\beta_0$ | Intercept |
| $\beta_1$ | Change in log-odds for a one-unit increase in $X$ |

Equation 4.2, p. 133. With several predictors, replace $\beta_0+\beta_1X$ by $\beta_0+\sum_{j=1}^{p}\beta_jX_j$ (Eqs. 4.6-4.7, p. 136).

### Odds and Logit

$$
\frac{p(X)}{1-p(X)}=e^{\beta_0+\beta_1X},
\qquad
\log\left(\frac{p(X)}{1-p(X)}\right)=\beta_0+\beta_1X
$$

| Symbol | Meaning |
|---|---|
| $p(X)/(1-p(X))$ | Odds of class 1 |
| $\log(p(X)/(1-p(X)))$ | Log-odds, or logit |
| $e^{\beta_1}$ | Multiplicative change in the odds for a one-unit increase in $X$ |

Equations 4.3-4.4, p. 134.

### Logistic Likelihood

$$
\ell(\beta_0,\beta_1)
=
\prod_{i:y_i=1}p(x_i)
\prod_{i':y_{i'}=0}\left[1-p(x_{i'})\right]
$$

| Symbol | Meaning |
|---|---|
| $\ell(\beta_0,\beta_1)$ | Likelihood of the observed training labels |
| $x_i$ | Predictors for observation $i$ |
| $y_i$ | Observed binary class for observation $i$ |
| $p(x_i)$ | Modelled probability that observation $i$ belongs to class 1 |

Equation 4.5, p. 135. The fitted coefficients maximize this likelihood.

### Multinomial Logistic Probability

$$
\Pr(Y=k\mid X=x)
=
\frac{e^{\beta_{k0}+\beta_{k1}x_1+\cdots+\beta_{kp}x_p}}
{\sum_{\ell=1}^{K}e^{\beta_{\ell0}+\beta_{\ell1}x_1+\cdots+\beta_{\ell p}x_p}}
$$

| Symbol | Meaning |
|---|---|
| $K$ | Number of response classes |
| $k$ | A particular class |
| $x_j$ | Value of predictor $j$ |
| $\beta_{kj}$ | Coefficient for predictor $j$ in class $k$ |

Equation 4.13, p. 139. This symmetric softmax form yields linear pairwise log-odds (Eq. 4.14, p. 140).

### Bayes' Theorem for Classification

$$
p_k(x)=\Pr(Y=k\mid X=x)
=
\frac{\pi_k f_k(x)}
{\sum_{\ell=1}^{K}\pi_\ell f_\ell(x)}
$$

| Symbol | Meaning |
|---|---|
| $p_k(x)$ | Posterior probability of class $k$ after observing $x$ |
| $\pi_k$ | Prior probability of class $k$ |
| $f_k(x)$ | Density of $X$ within class $k$ |
| $K$ | Number of classes |

Equation 4.15, p. 147. Generative classifiers differ mainly in how they estimate $f_k(x)$.

### LDA Discriminant Function

$$
\delta_k(x)
=
x^T\Sigma^{-1}\mu_k
-\frac{1}{2}\mu_k^T\Sigma^{-1}\mu_k
+\log(\pi_k)
$$

| Symbol | Meaning |
|---|---|
| $\delta_k(x)$ | Discriminant score for class $k$ |
| $x$ | Predictor vector for the new observation |
| $\mu_k$ | Mean predictor vector for class $k$ |
| $\Sigma$ | Covariance matrix shared by all classes |
| $\pi_k$ | Prior probability of class $k$ |
| $T$ | Matrix transpose |

Equation 4.24, p. 151. LDA predicts the class with the largest $\delta_k(x)$; because the quadratic terms shared across classes cancel, its decision boundaries are linear.

### QDA Discriminant Function

$$
\delta_k(x)
=
-\frac{1}{2}(x-\mu_k)^T\Sigma_k^{-1}(x-\mu_k)
-\frac{1}{2}\log|\Sigma_k|
+\log(\pi_k)
$$

| Symbol | Meaning |
|---|---|
| $\delta_k(x)$ | Discriminant score for class $k$ |
| $\mu_k$ | Mean vector for class $k$ |
| $\Sigma_k$ | Covariance matrix specific to class $k$ |
| $|\Sigma_k|$ | Determinant of the class covariance matrix |
| $\pi_k$ | Prior probability of class $k$ |

Equation 4.28, p. 160. Class-specific covariance matrices preserve quadratic terms and therefore allow curved boundaries.

### Naive Bayes Conditional Independence

$$
f_k(x)=\prod_{j=1}^{p}f_{kj}(x_j)
$$

| Symbol | Meaning |
|---|---|
| $f_k(x)$ | Joint predictor density within class $k$ |
| $f_{kj}(x_j)$ | Density of predictor $j$ within class $k$ |
| $p$ | Number of predictors |

Equation 4.29, p. 162. It assumes predictors are independent conditional on the class; substituting this product into Bayes' theorem gives Eq. 4.30 (p. 163).

### Poisson Regression Mean

$$
\log\left(\lambda(X_1,\ldots,X_p)\right)
=
\beta_0+\beta_1X_1+\cdots+\beta_pX_p
$$

$$
\lambda(X_1,\ldots,X_p)
=
e^{\beta_0+\beta_1X_1+\cdots+\beta_pX_p}
$$

| Symbol | Meaning |
|---|---|
| $Y$ | Non-negative integer count response |
| $\lambda(X)$ | Conditional mean $E(Y\mid X)$ and Poisson rate |
| $X_j$ | Predictor $j$ |
| $\beta_j$ | Change in log expected count per unit increase in $X_j$ |

Equations 4.36-4.37, p. 170. Together with the Poisson probability mass function in Eq. 4.35 (p. 169), these define Poisson regression.

## Intuition

Imagine a hospital triage desk deciding which department should see a patient. Logistic regression is like a scoring checklist: each symptom adds or subtracts from a score, and the score is converted into a probability. LDA, QDA, and naive Bayes instead imagine profiles for patients already known to belong to each department, then ask which profile could most plausibly have produced the new patient. LDA assumes the departments have similarly shaped clouds of patient characteristics; QDA allows each cloud to have a different shape; naive Bayes treats the symptoms as independent once the correct department is known. KNN simply looks at the most similar past patients and follows their labels. The chosen probability threshold expresses the cost of mistakes: if missing an emergency is dangerous, triage can use a lower threshold and accept more false alarms.

## Statistical Assumptions

- **Independent observations:** Logistic regression, LDA, QDA, naive Bayes, and their standard errors normally assume training observations are independent. Dependence can make uncertainty estimates too optimistic and let related observations leak information across train/test splits.
- **Correct linear log-odds for logistic regression:** Binary and multinomial logistic regression assume each relevant log-odds relationship is linear in the supplied features. Non-linearity or omitted interactions can create biased probabilities and poor boundaries; transformations, interaction terms, QDA, KNN, trees, or other flexible methods may work better.
- **No perfect multicollinearity:** Logistic regression requires the design matrix to contain enough independent information to estimate its coefficients. Exact redundancy makes coefficients unidentified, while strong correlation makes estimates unstable.
- **No complete or quasi-complete separation:** If predictors perfectly separate the classes, unregularized logistic coefficient estimates can diverge. Penalized logistic regression or more data is then needed.
- **Representative class priors:** Generative classifiers use $\pi_k$ to represent class prevalence. Training samples collected with distorted class proportions require corrected priors or the resulting posterior probabilities and decisions will be miscalibrated.
- **Multivariate normality for LDA and QDA:** Both methods model predictors within each class as Gaussian. Severe skew, heavy tails, or multimodality can make their estimated densities and boundaries misleading.
- **Equal covariance matrices for LDA:** LDA assumes all classes share $\Sigma$. If class spreads or correlations differ substantially, QDA can reduce bias when enough data are available.
- **Enough data for QDA:** QDA estimates a full covariance matrix for every class. With many predictors or few observations, those estimates become noisy or singular, producing high variance.
- **Conditional independence for naive Bayes:** Within a class, naive Bayes assumes predictors are independent. Strong remaining dependencies can distort posterior probabilities, though classification may still be good because the assumption substantially lowers variance.
- **Local smoothness and meaningful distance for KNN:** Nearby observations should tend to share a class, predictors should be scaled comparably, and irrelevant dimensions should be controlled. Otherwise distance becomes misleading, especially in high dimensions.
- **Poisson mean-variance equality:** Basic Poisson regression assumes $\operatorname{Var}(Y\mid X)=E(Y\mid X)$. Overdispersion leads to underestimated standard errors; quasi-Poisson or negative-binomial models are common alternatives.

## Decision Guide

| Situation | Use this chapter's method | Use instead |
|---|---|---|
| Binary outcome; interpretable effects and probabilities matter | Logistic regression | A flexible classifier if the log-odds boundary is strongly non-linear |
| More than two unordered classes | Multinomial logistic regression | Ordered logistic regression when classes have a natural order |
| Classes are roughly Gaussian with similar covariance | LDA | QDA if covariance structures differ |
| Boundary is non-linear and each class has enough observations | QDA | LDA or regularized methods when data are limited |
| Many predictors and a useful conditional-independence approximation | Naive Bayes | Logistic regression when interactions and calibrated probabilities matter |
| Boundary is irregular, dimension is modest, and there is ample data | KNN with tuned $K$ | A parametric method in high dimensions or when interpretation matters |
| False negatives and false positives have unequal costs | Tune the probability threshold; compare sensitivity and specificity | Do not rely on the default 0.5 threshold or accuracy alone |
| Compare ranking ability across all thresholds | ROC curve and AUC | Precision-recall analysis when the positive class is very rare |
| Non-negative count response | Poisson regression | Negative binomial or quasi-Poisson when counts are overdispersed |

## Python Implementation

```python
import numpy as np
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.impute import SimpleImputer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, confusion_matrix, roc_auc_score
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler

# Example DataFrame:
# df = pd.read_csv("data.csv")
# target is binary: 0 = No, 1 = Yes
X = df.drop(columns="target")
y = df["target"]

numeric_features = X.select_dtypes(include=np.number).columns
categorical_features = X.select_dtypes(exclude=np.number).columns

preprocess = ColumnTransformer(
    transformers=[
        (
            "num",
            Pipeline([
                ("imputer", SimpleImputer(strategy="median")),
                ("scale", StandardScaler()),
            ]),
            numeric_features,
        ),
        (
            "cat",
            Pipeline([
                ("imputer", SimpleImputer(strategy="most_frequent")),
                ("encode", OneHotEncoder(handle_unknown="ignore")),
            ]),
            categorical_features,
        ),
    ]
)

model = Pipeline([
    ("preprocess", preprocess),
    ("classifier", LogisticRegression(
        penalty="l2",       # regularizes coefficients
        C=1.0,              # smaller values mean stronger regularization
        max_iter=1000,      # allow the optimizer enough iterations
        class_weight=None,  # use "balanced" cautiously for imbalanced classes
        random_state=42,
    )),
])

model.fit(X_train, y_train)
probability = model.predict_proba(X_test)[:, 1]

threshold = 0.30  # choose using validation data and error costs
prediction = (probability >= threshold).astype(int)

print(confusion_matrix(y_test, prediction))
print(classification_report(y_test, prediction))
print("ROC AUC:", roc_auc_score(y_test, probability))
```

Fit preprocessing and threshold selection using training/validation data only; reserve the test set for final evaluation.

## Common Pitfalls

- ⚠️ **Using linear regression for a categorical response** - It can produce probabilities outside $[0,1]$ and imposes arbitrary numeric spacing between response categories.
- ⚠️ **Interpreting $\beta_j$ as a direct probability change** - In logistic regression, $\beta_j$ changes log-odds and $e^{\beta_j}$ multiplies odds; the probability change depends on all predictors and the starting probability.
- ⚠️ **Confusing statistical significance with predictive usefulness** - A small coefficient can be statistically significant in a large sample yet add little out-of-sample discrimination.
- ⚠️ **Treating 0.5 as a universally correct threshold** - The optimal threshold depends on class prevalence and the relative costs of false positives and false negatives.
- ⚠️ **Reporting accuracy for an imbalanced problem** - A model can appear accurate by always predicting the common class; inspect the confusion matrix, sensitivity, specificity, and a suitable curve-based metric.
- ⚠️ **Selecting a threshold on the test set** - This leaks test information and makes performance optimistic. Tune it with training or validation data.
- ⚠️ **Assuming a high AUC means calibrated probabilities** - AUC measures ranking, not whether a predicted probability of 0.8 occurs about 80% of the time.
- ⚠️ **Using QDA with too little data** - Estimating one covariance matrix per class can cause severe variance or singular matrices.
- ⚠️ **Taking "naive" independence literally** - Naive Bayes may classify well with correlated predictors, but its probability estimates can be overconfident.
- ⚠️ **Applying KNN before scaling predictors** - Variables with large numeric ranges dominate distance; imputation and standardization must be learned within the training pipeline.
- ⚠️ **Ignoring overdispersion in Poisson regression** - When count variance greatly exceeds its mean, standard errors and inference from the basic Poisson model are unreliable.

## Connections to Other Chapters

- **Builds on:** [Ch02 - Statistical Learning](./Ch02%20-%20Statistical%20Learning.md) - Uses the Bayes classifier, bias-variance trade-off, test error, and flexible versus inflexible modelling ideas introduced in Chapter 2.
- **Builds on:** [Ch03 - Linear Regression](./Ch03%20-%20Linear%20Regression.md) - Reuses linear predictors, coefficient interpretation, dummy variables, interactions, maximum-likelihood-style inference, and model diagnostics for non-quantitative responses.
- **Leads to:** [Ch05 - Resampling Methods](./Ch05%20-%20Resampling%20Methods.md) - Cross-validation provides a principled way to compare classifiers, tune $K$, regularization, and probability thresholds, and estimate test error.
- **Leads to:** [Ch08 - Tree-Based Methods](./Ch08%20-%20Tree-Based%20Methods.md) - Trees and ensembles provide flexible alternatives when decision boundaries contain complex non-linearities and interactions.
- **Leads to:** [Ch09 - Support Vector Machines](./Ch09%20-%20Support%20Vector%20Machines.md) - Support vector classifiers construct flexible classification boundaries, especially in high-dimensional settings.

## Review Questions

**Q1:** Why is logistic regression generally preferred to linear regression for a binary response?

> [!note]- Answer
> Logistic regression maps a linear predictor through the logistic function, so fitted values are valid probabilities between 0 and 1. It also models log-odds linearly and uses a likelihood designed for binary data, whereas linear regression can produce impossible probabilities and assumes an inappropriate constant-variance Gaussian error model.

**Q2:** What is the key difference between LDA and QDA, and how does it affect bias and variance?

> [!note]- Answer
> LDA assumes all classes share one covariance matrix, which creates linear decision boundaries and requires fewer parameters. QDA estimates a separate covariance matrix for each class, allowing quadratic boundaries and lower bias when covariances truly differ, but its extra parameters increase variance and require more data.

**Q3:** Why can changing the classification threshold improve a model for a real decision without changing the fitted model itself?

> [!note]- Answer
> The fitted model supplies posterior probabilities, while the threshold converts those probabilities into actions. Lowering the threshold predicts more positives, usually increasing sensitivity and false positives; raising it does the reverse. The best action threshold therefore depends on the relative cost of each error and can differ from 0.5 even though the probabilities stay unchanged.

## My Notes & Observations


---
*[00 - Index](../00%20-%20Index.md) | [[Labs/Lab04 - Classification]] | Next: [[Chapters/Ch05 - Resampling Methods]]*
---
