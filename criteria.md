# Acceptance Criteria — TakeMeter

These five criteria define what I consider a successful mathematics question classifier. I chose the targets before training so I can evaluate the model against them later.

TakeMeter will fine-tune `distilbert-base-uncased` to classify Mathematics Stack Exchange questions into three categories: `proof_help`, `computation`, and `conceptual`.

I collected 500 candidate questions and plan to use at least 200 human-reviewed examples for training and evaluation.

**How the criteria will be evaluated:**

Criteria 1, 2, 3, and 5 will be measured using the held-out test set from the notebook's 70/15/15 split. In Unit 6, each criterion must pass across all three random seeds (42, 7, and 2024).

Criterion 4 will be checked using the entire labeled dataset before splitting.

---

## 1. Overall Accuracy of at Least 0.70

The classifier must correctly categorize at least **70% of the questions** in the held-out test set for each seed.

**How to check:** Use the `accuracy` value in `results.json` and `results_three_seeds_before.json`. Each run must have an accuracy of at least `0.70`.

**Why this target:**

I chose 70% because correctly identifying the type of mathematical help in seven out of ten questions would make the classifier useful for basic question sorting. Some questions may be difficult to classify, especially those that ask for both proofs and conceptual explanations. With three categories and a maximum class share of 60%, this target is also higher than simply predicting the most common label in the full dataset. I will compare it with the actual majority-label share in each test split.

---

## 2. Macro F1 Score of at Least 0.65

The classifier must achieve a **macro F1 score of at least 0.65** on the held-out test set for each seed.

**How to check:** Use the `f1_macro` value returned by the notebook's `evaluate()` function. Each run must achieve at least `0.65`.

**Why this target:**

I chose 0.65 because I want the model to perform reasonably well across all three categories, not just the most common one. Macro F1 gives each label equal importance. This target also allows for some mistakes because we are training on a relatively small dataset of around 200 questions.

---

## 3. F1 Score of at Least 0.60 for Every Label

Each label (`proof_help`, `computation`, and `conceptual`) must achieve an **F1 score of at least 0.60** on the held-out test set for each seed.

If any label falls below 0.60 in any run, this criterion is not met.

**How to check:** Use `per_label[label]["f1"]` from the notebook's evaluation results. Also record the number of test examples for each label using `per_label[label]["support"]`.

**Why this target:**

I chose 0.60 because I don't want the classifier to perform well overall while struggling with one category. For example, `proof_help` and `conceptual` may be confused because both types of questions can include words like "why" or "explain." This target makes sure each label has acceptable performance while accounting for our small test set.

---

## 4. Each Label Represents Between 20% and 60% of the Dataset

Each of the three labels must account for **at least 20% and no more than 60%** of the final `labels.csv` dataset.

The dataset must also contain at least 200 human-reviewed questions.

**How to check:** Count the examples for each label and divide by the total number of examples. Every label must have a share between `0.20` and `0.60`, inclusive.

**Why this target:**

I chose this range to prevent one category from dominating the dataset. With 200 questions, the 20% minimum ensures that each category has at least 40 examples. The 60% maximum is also stricter than CodePath's required 70% limit. This should help the model learn all three categories instead of mainly predicting the most common one.

---

## 5. More Confident Predictions Should Be More Accurate

For each seed, the **most-confident third of predictions must have an accuracy of at least 0.80**.

Their accuracy must also be **at least 0.10 (10 percentage points) higher** than the least-confident third.

Both conditions must be met.

**How to check:** Use the values from the notebook's `confidence_report()` function:

- `accuracy_most_confident_third >= 0.80`
- `accuracy_most_confident_third - accuracy_least_confident_third >= 0.10`

Compare the unrounded values, allowing a small floating-point tolerance of `1e-12` for the difference.

**Why this target:**

I chose these targets because the model's confidence should help identify which questions may need human review. If the model is highly confident, I would expect its predictions to be more reliable. This could be especially useful for questions that fall between `proof_help` and `conceptual`. However, with only about 30 test questions, each confidence group will be small, so the results may vary between runs.

---

<!--
UNIT 6 — IMPORTANT

Keep the original criteria and targets unchanged when evaluating the model.

If a criterion is not met, explain why and attempt an improvement.
Do not lower a target simply because the model failed to reach it.

If a criterion cannot actually be measured as written, you may add a
clarified version underneath the original criterion. Explain why the
revision was necessary.

Example:

> Revised in Unit 6: Every label must achieve an F1 score of at least
> 0.60 on the held-out test set.
>
> Why revised: The original wording was too vague to measure.

A missed target is not the same as a broken criterion.
-->
