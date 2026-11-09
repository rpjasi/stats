# Principal Components
> **Category:** Dimension reduction concept
> **First appears:** [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md) (p. 4)
> **Back to glossary:** [00 - Glossary](00 - Glossary.md)

## Plain English Definition
Principal components are new variables formed from the original variables to summarize the dominant variation in high-dimensional data. In Chapter 1, the first two principal components, $Z_1$ and $Z_2$, provide a two-dimensional view of the NCI60 gene-expression data.

## Formal Definition
$$
Z_m = X\phi_m
$$
| Symbol | Meaning |
|---|---|
| $Z_m$ | The $m$th principal component score |
| $X$ | Feature matrix |
| $\phi_m$ | Direction defining the $m$th principal component |
| $m$ | Component index |

## Intuitive Analogy
Principal components are like finding the best camera angles for a complicated 3D object. Instead of staring at every possible direction at once, you choose the views that reveal the most important shape with the least clutter.

## When to Use It
- ✅ When you need a compact summary of many correlated variables for visualization or preprocessing.

## When It Breaks Down
- ❌ When the important structure is nonlinear or when reducing dimension discards details that matter for the task.

## Related Terms
- [Unsupervised Learning](Unsupervised Learning.md) — principal components are commonly used in unsupervised analysis
- [Feature Matrix](Feature Matrix.md) — principal components are computed from the predictor matrix

## Seen In
- [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md)
- [Ch12 - Unsupervised Learning](../Chapters/Ch12 - Unsupervised Learning.md)

---
*[00 - Glossary](00 - Glossary.md) | [00 - Index](../00 - Index.md)*
