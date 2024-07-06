---
layout: post
title:  "Compute accuracy from F1 score"
date:   2024-07-06 09:51:59 +0800
tags:   ml
---

I encountered a similar problem today as the one in [this](https://stackoverflow.com/questions/42041078/calculating-accuracy-from-precision-recall-f1-score-scikit-learn) post, where I wish to find the accuracy given F1 score only.
F1 score is [well suited](https://datascience.stackexchange.com/a/65342/153995) to my imbalanced classification problem, so I compute it during training; but I then find it difficult to interprete.
There's a surprising lack of relevant information when I searched the web.
Luckily, it's not a difficult task either.

Since each F1 score corresponds to a range of accuracies, we may regard finding the accuracy given F1 score an optimization problem.
The process consists of two steps: 1) find the minimum accuracy; 2) find the maximum accuracy. To find the maximum, we may reduce it to finding the *negative* of the minimum of the *negative* accuracy.
Thus we will only handle step 1 below.

Known constants:

- $s_F$: the F1 score.
- $r_P$ and $r_N$: the positive and negative class ratio.

Variables:

- $r_{TP}$, $r_{TN}$, $r_{FP}$, $r_{FN}$: the true positive, true negative, false positive and false negative ratio (i.e. divided by the total sample count).

Objective:
$s_A = r_{TP} + r_{TN}$.

Constraints:

- $r_{TP} \ge 0$, $r_{TN} \ge 0$, $r_{FP} \ge 0$, $r_{FN} \ge 0$.
- $r_{TP} + r_{FN} = r_P$, $r_{TN} + r_{FP} = r_N$.
- $\frac{2 \cdot r_{TP} / (r_{TP} + r_{FP}) \cdot r_{TP} / (r_{TP} + r_{FN})}{r_{TP} / (r_{TP} + r_{FP}) + r_{TP} / (r_{TP} + r_{FN})} = s_F$. The left hand side is just the F1 score formula.

Python implementation:

```python
# jax is not necessary, just that I don't want to spend time on finding
# partial derivative of the F1 score with respect to true positive,
# etc.
import jax
import numpy as np
from scipy.special import softmax
from scipy.optimize import minimize

# Used to avoid divid-by-zero error.
EPS = 1e-8

def f1_score_constraint(x, f1_score):
    """
    :param x: the array (tp, fp, tn, fn)
    :param f1_score: the known F1 score
    """
    tp, fp, fn = x[0], x[2], x[3]
    precision = tp / (tp + fp)
    recall = tp / (tp + fn)
    return 2 * (precision * recall) / (precision + recall) - f1_score


def positive_sum_constraint(x, n_positive):
    """
    :param x: the array (tp, fp, tn, fn)
    :param n_positive: the known positive class ratio
    """
    tp, fn = x[0], x[3]
    return tp + fn - n_positive


def negative_sum_constraint(x, n_negative):
    """
    :param x: the array (tp, fp, tn, fn)
    :param n_negative: the known negative class ratio
    """
    tn, fp = x[1], x[2]
    return tn + fp - n_negative


def accuracy(x):
    """
    :param x: the array (tp, fp, tn, fn)
    """
    tp, tn = x[0], x[1]
    return tp + tn


# Ideally this should give a feasible solution. But in practice, I
# find it works fine even if it's not feasible.
def rand_init():
    return softmax(np.random.randn(4))


def find_min_accuracy_from_f1(f1_score, n_positive, n_negative):
    """
    :param f1_score: the known F1 socre
    :param n_positive: the known positive class ratio
    :param n_negative: the known negative class ratio
    """
    res = minimize(
        accuracy,
        rand_init(),
        method='SLSQP',
        jac=jax.grad(accuracy),
        bounds=[(EPS, None), (EPS, None), (EPS, None), (EPS, None)],
        constraints=[
            {
                'type': 'eq',
                'fun': f1_score_constraint,
                'jax': jax.grad(f1_score_constraint),
                'args': (f1_score,),
            },
            {
                'type': 'eq',
                'fun': positive_sum_constraint,
                'jac': jax.grad(positive_sum_constraint),
                'args': (n_positive,),
            },
            {
                'type': 'eq',
                'fun': negative_sum_constraint,
                'jac': jax.grad(negative_sum_constraint),
                'args': (n_negative,),
            },
        ],
        options={'maxiter': 1000},
    )
    return res.fun
```

Calling the function `find_min_accuracy_from_f1` with data, we get the minimum possible accuracy given F1 score:

```
>>> find_min_accuracy_from_f1(0.457, 0.044, 0.9559)
0.8953
```
