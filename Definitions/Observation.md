# Observation
> **Category:** Data representation
> **First appears:** [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md) (p. 8)
> **Back to glossary:** [00 - Glossary](./00 - Glossary.md)

## Plain English Definition
An observation is one sampled unit in the dataset, such as one person, one day, or one cell line. In the textbook notation, observations are indexed by $i=1,\dots,n$.

## Formal Definition
$$
i \in \{1,2,\dots,n\}
$$
| Symbol | Meaning |
|---|---|
| $i$ | Index for an observation |
| $n$ | Total number of observations |

## Intuitive Analogy
An observation is like one row in a spreadsheet for a single case. If the spreadsheet is a class register, then each row corresponds to one student and all the measurements recorded for that student.

## When to Use It
- ✅ When describing data row by row or indexing units in a sample.

## When It Breaks Down
- ❌ When the rows do not correspond to comparable units or when repeated measurements are treated as independent without care.

## Related Terms
- [Variable](./Variable.md) — variables are the columns measured for each observation
- [Feature Matrix](./Feature Matrix.md) — observations form the rows of the feature matrix

## Seen In
- [Ch01 - Introduction](../Chapters/Ch01 - Introduction.md)
- [Ch02 - Statistical Learning](../Chapters/Ch02 - Statistical Learning.md)

---
*[00 - Glossary](./00 - Glossary.md) | [00 - Index](../00 - Index.md)*
