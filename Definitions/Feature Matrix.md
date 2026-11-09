# Feature Matrix
> **Category:** Data representation
> **First appears:** [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md) (p. 9)
> **Back to glossary:** [00 - Glossary](00 - Glossary.md)

## Plain English Definition
The feature matrix collects all predictor values into one $n \times p$ array. Each row is an observation, each column is a variable, and the entry in row $i$, column $j$ is $x_{ij}$.

## Formal Definition
$$
X =
\begin{pmatrix}
x_{11} & x_{12} & \cdots & x_{1p} \\
x_{21} & x_{22} & \cdots & x_{2p} \\
\vdots & \vdots & \ddots & \vdots \\
x_{n1} & x_{n2} & \cdots & x_{np}
\end{pmatrix}
$$
| Symbol | Meaning |
|---|---|
| $X$ | Feature matrix |
| $x_{ij}$ | Value of variable $j$ for observation $i$ |
| $n$ | Number of observations |
| $p$ | Number of variables |

## Intuitive Analogy
The feature matrix is the full spreadsheet of predictors. If you imagine data as a table pinned to a wall, the matrix is the complete grid that every algorithm reads when it tries to learn from the inputs.

## When to Use It
- ✅ When expressing statistical learning methods in compact matrix form.

## When It Breaks Down
- ❌ When the predictors are not aligned consistently across rows and columns or when important data structure is lost by forcing everything into a simple rectangular table.

## Related Terms
- [Observation](Observation.md) — observations correspond to rows of $X$
- [Variable](Variable.md) — variables correspond to columns of $X$
- [Response Vector](Response Vector.md) — the response vector pairs with $X$ in supervised learning

## Seen In
- [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md)
- [Ch03 - Linear Regression](../Chapters/Ch03 - Linear Regression.md)

---
*[00 - Glossary](00 - Glossary.md) | [00 - Index](../00 - Index.md)*
