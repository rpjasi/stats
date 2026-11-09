# Clustering
> **Category:** Unsupervised learning task
> **First appears:** [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md) (p. 4)
> **Back to glossary:** [00 - Glossary](00 - Glossary.md)

## Plain English Definition
Clustering is the task of grouping observations so that similar observations fall into the same group. In Chapter 1, it is motivated by grouping cancer cell lines using gene-expression profiles.

## Formal Definition
$$
\mathcal{C} = \{C_1, C_2, \dots, C_G\}
$$
| Symbol | Meaning |
|---|---|
| $\mathcal{C}$ | A partition of the observations into clusters |
| $C_g$ | The $g$th cluster |
| $G$ | Number of clusters |

## Intuitive Analogy
Clustering is like organizing a pile of photographs without captions. You spread them out on a table and place visually similar images together, hoping the groups reflect meaningful underlying categories.

## When to Use It
- ✅ When you have unlabeled data and want to detect natural groupings or segments.

## When It Breaks Down
- ❌ When the data have no real group structure, or when the number and shape of groups depend too heavily on arbitrary modeling choices.

## Related Terms
- [Unsupervised Learning](Unsupervised Learning.md) — clustering is a central unsupervised task
- [Principal Components](Principal Components.md) — often used before clustering to reduce dimension

## Seen In
- [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md)
- [Ch12 - Unsupervised Learning](../Chapters/Ch12 - Unsupervised Learning.md)

---
*[00 - Glossary](00 - Glossary.md) | [00 - Index](../00 - Index.md)*
