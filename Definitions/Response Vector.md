# Response Vector
> **Category:** Data representation
> **First appears:** [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md) (p. 10)
> **Back to glossary:** [00 - Glossary](00 - Glossary.md)

## Plain English Definition
The response vector contains the observed outcome for each observation in a supervised learning problem. It is the target that a model tries to predict from the feature matrix.

## Formal Definition
$$
\mathbf{y} =
\begin{pmatrix}
y_1 \\
y_2 \\
\vdots \\
y_n
\end{pmatrix}
$$
| Symbol | Meaning |
|---|---|
| $\mathbf{y}$ | Response vector |
| $y_i$ | Response for observation $i$ |
| $n$ | Number of observations |

## Intuitive Analogy
The response vector is like the answer column in a training worksheet. You keep the questions in one table and stack the correct outcomes in a separate column so the model can learn the relationship between the two.

## When to Use It
- ✅ When formulating supervised learning problems with a single outcome per observation.

## When It Breaks Down
- ❌ When no response exists, when the labels are unreliable, or when the outcome is not aligned correctly with the rows of the feature matrix.

## Related Terms
- [Supervised Learning](Supervised Learning.md) — the response vector is what makes the setting supervised
- [Feature Matrix](Feature Matrix.md) — the response vector is modeled as a function of the feature matrix

## Seen In
- [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md)
- [Ch03 - Linear Regression](../Chapters/Ch03 - Linear Regression.md)

---
*[00 - Glossary](00 - Glossary.md) | [00 - Index](../00 - Index.md)*
