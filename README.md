# TakeMeter


<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter is a text classifier for questions posted on Mathematics Stack Exchange. From 500 public questions, I selected and labeled 200 examples according to the type of mathematical help the author is requesting: `proof_help`, `computation`, or `conceptual`. I fine-tuned `distilbert-base-uncased` on this dataset and evaluated it on held-out questions. Dataset preparation, human review, and the initial training run are recorded below.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `proof_help`

**Definition:**
A question whose primary objective is to prove or disprove a mathematical claim, construct a counterexample, or verify or correct the logical validity of a proof.

**Example 1:** 
> Prove that the Cauchy product of two absolutely convergent series converges absolutely 

The author presents a proof attempt and asks for help justifying the argument.

**Example 2:**
> What is wrong with this proof of commutativity of vector addition? 

The author asks which step in an existing mathematical proof is invalid.

### `computation`

**Definition:**
A question whose primary objective is to calculate, simplify, evaluate, or solve a specified mathematical expression, equation, or quantity.

**Example 1:**
>How to calculate a type of alternating sums

The author wants to evaluate an alternating series and understand how to calculate sums of that form.

**Example 2:**
>Solving an equation involving floor and fractional parts

The author asks to solve an equation and determine a specific numerical expression from its possible solutions.

### `conceptual`

**Definition:**
A question whose primary objective is to understand the meaning, interpretation, intuition, or motivation behind a mathematical concept rather than obtain a proof or computed result.

**Example 1:**
>Why not think of lattices as Zⁿ?

The author wants to understand why lattices are treated as abstract mathematical objects rather than simply as Zⁿ.

**Example 2:**
>Conceptualizing subgaussian norms

The author asks for intuition about subgaussian norms and how to interpret their mathematical meaning.


### The hardest boundary

**Which two labels:** proof_help and conceptual

**The decision rule I used every time:**
<!-- e.g. "If the post names a specific checkable fact, it's `analysis`, even
     if the tone is heated." -->
If the main goal of a question is to construct, verify, correct, or disprove a formal mathematical argument, I classify it as proof_help, even if the author uses words such as "explain" or "why." If the author mainly wants to understand the meaning or intuition behind a concept without requesting a formal proof, I classify it as conceptual.

For example, the question asking what is wrong with a proof of vector addition belongs to proof_help because the author wants to identify an invalid step in a specific argument, not simply understand what vector addition means.

For questions that could fall between computation and proof_help, I use the requested outcome to decide: finding a particular solution or value is computation, while proving that a mathematical statement holds is proof_help.

Questions primarily requesting references, discussing community moderation, or asking about broader mathematical research topics are outside the scope of these three labels. I will apply this exclusion rule consistently when selecting the training examples.


---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:**

I downloaded 500 real public Mathematics Stack Exchange questions through Stack Exchange Data Explorer. The original export is saved as [data/Query_Results_From_Math_Stack.csv](data/Query_Results_From_Math_Stack.csv). The selected 200 questions are in [labels.csv](labels.csv) at the repository root, with exactly `text`, `label`, and `note` columns.

Each example includes its question title and body text, with mathematical notation preserved and HTML formatting removed. The questions were not shortened or generated by AI. [data/math_selected_sources.csv](data/math_selected_sources.csv) records the source ID, question URL, and exported content license (CC BY-SA 4.0) for each selected question, in the same row order as `labels.csv`.

**How I labelled them:** <!-- Cold first? Pre-labelled with AI and corrected?
Say so plainly — the disclosure is required, not penalised. -->

I labeled the first 20 questions independently, without AI assistance, and marked their notes `cold`. I used AI suggestions for the remaining 180 questions and reviewed their labels against the taxonomy above. I classified each question by the author's primary requested outcome, rather than by its subject or words such as "why," "proof," or "calculate."

An additional AI-assisted audit led to 38 label corrections and six replacements for questions outside the classifier's scope, including reference requests and software troubleshooting. The replacements came from unused questions in the same original 500-question export. I approved six corrections in small batches, then delegated the remaining assisted audit decisions. Afterward, I reviewed and confirmed the other 32 corrected labels and all six replacement labels. Their notes preserve both AI assistance and my review confirmation.

The original 20 `cold` records remain unchanged. Their original independent annotations are preserved in [original_cold_annotations.json](data/label_audit/original_cold_annotations.json).


**Counts per label:**

| Label | Count | Share |
|---|---:|---:|
| `proof_help` | 94 | 47% |
| `computation` | 40 | 20% |
| `conceptual` | 66 | 33% |
| **Total** | **200** | **100%** |

Every label meets Criterion 4's inclusive 20%–60% range and the assignment's 70% maximum. The dataset remains one unsplit file; the notebook created the stratified training, validation, and test split using its default 70/15/15 procedure. The actual split counts are recorded below.

Validation confirmed 200 unique question texts and source IDs, no missing text, labels, or notes, and only the three allowed labels. The CSV loads with pandas, and Section 2's `check_dataset()` function returns no problems or warnings. These checks validate the dataset's structure; they do not resolve the five cold-label flags. Details are in [final_validation.json](data/label_audit/final_validation.json).

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* [Proof by contradiction and logical value](https://math.stackexchange.com/questions/5152102)
>
> *Could have been:* `proof_help` or `conceptual`.
>
> *I chose, because:* `conceptual`. Although the question discusses a proof by contradiction, the author asks how to interpret the truth value of a contradiction under an assumed hypothesis. The primary outcome is understanding the logic, rather than constructing or checking a particular proof. I approved changing its original `proof_help` label.

**2.**
> *The post:* [A polynomial lower bound for $-x\ln x-y\ln y-z\ln z$](https://math.stackexchange.com/questions/5149777)
>
> *Could have been:* `computation` or `proof_help`.
>
> *I chose, because:* `proof_help`. The expressions could suggest a calculation task, but the final request explicitly asks whether the proposed inequality can be proved for all positive $x,y,z$ under the stated constraint. A general proof is the requested outcome. I confirmed the correction from `computation`.

**3.**
> *The post:* [Ratio of areas of related triangles in an intuitive fashion](https://math.stackexchange.com/questions/5151138)
>
> *Could have been:* `conceptual` or `proof_help`.
>
> *I chose, because:* `proof_help`. The title asks for an intuitive approach, but the post explicitly asks to prove a triangle-area identity and seeks alternative or shorter proofs. The request for intuition describes the preferred proof style. I confirmed the correction from `conceptual`.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`.

**Settings:** <!-- epochs, learning rate, batch size, seed -->

| Setting | Value |
|---|---|
| Epochs | 3 |
| Learning rate | `2e-5` |
| Batch size | 16 for training and evaluation |
| Maximum sequence length | 128 tokens, with truncation and padding |
| Random seed | 42 |
| Hardware | Apple Silicon GPU (MPS) |
| PyTorch | 2.14.1 |


**Anything I changed from the defaults, and why:**

I replaced the sample label map with `proof_help`, `computation`, and `conceptual` to match my mathematical assistance taxonomy, and used the 200 questions in root `labels.csv`. I also filled in the notebook's name and email fields. I kept the base model, hyperparameters, seed, splitting procedure, and training code at their original defaults.

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->

| Label | Train | Validation | Test |
|---|---:|---:|---:|
| `proof_help` | 65 | 15 | 14 |
| `computation` | 28 | 6 | 6 |
| `conceptual` | 46 | 10 | 10 |
| **Total** | **139** | **31** | **30** |

The `computation` category has only six test examples, so its evaluation metrics may be unstable across different splits or random seeds. These are the actual counts printed by the notebook; I did not manually split or change the dataset.

**Initial outcome:** [results.json](results.json) records test accuracy of 0.467 and macro F1 of 0.212. Per-label F1 is 0.636 for `proof_help` and 0.000 for both `computation` and `conceptual`. Its confusion counts confirm that the model predicted `proof_help` for all 30 test examples. Validation loss decreased across the three epochs (1.012956, 0.996696, 0.987934), while validation accuracy stayed at approximately 48.4%.

The completed run saved both `results.json` and the 30 held-out examples in [test_split.csv](test_split.csv). These initial results do not establish that the model meets the acceptance criteria. The targets remain unchanged; detailed diagnosis, baseline comparisons, improvements, and evaluation across three seeds belong to Unit 6.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* An audit of all 200 labels using the README's definitions and boundary rules, while preserving the original 20 cold annotations.
- *What came back:* Proposed corrections with explanations based on the author's primary requested outcome, including conceptual questions about proofs and proof requests initially labeled as computations.
- *What I changed:* I approved corrections in small batches and later delegated the remaining assisted audit decisions. The audit applied 38 label corrections in total. I then reviewed and confirmed the 32 corrected labels that had not already received my explicit approval. The cold records were preserved, with their five audit flags still documented.

**Moment 2**

- *What I asked for:* Real replacements for out-of-scope questions, using the existing math collection and keeping the training dataset in root `labels.csv`.
- *What came back:* Six unused questions from the original export, with source IDs, URLs, and suggested `computation` labels.
- *What I changed:* The six out-of-scope assisted records were replaced programmatically. I personally reviewed and confirmed all six replacement labels. The dataset stayed at 200 questions, and the replacement notes retain their AI-assisted provenance.

**Unit 6, Milestone 1 assistance:** I ran the baseline myself and supplied the prediction I had written before that run. AI helped verify that the saved baseline predictions match all 30 original held-out questions, check the score calculations, and format and explain the comparison below. It did not rerun either model or change the dataset, definitions, acceptance criteria, or Unit 5 results.

**Unit 6, Milestone 2 assistance:** After my VS Code run stalled and I reset the notebook, AI executed the notebook's setup and three-seed experiment in a fresh local IPython process, saved the notebook outputs, checked the result calculations, and prepared the run log and confusion matrix below. An initial attempt stalled under memory pressure; the completed run releases each seed's model and clears unused MPS cache before starting the next seed. This is execution cleanup, not the measured model improvement for Milestone 5. The training functions, hyperparameters, labels, taxonomy, criteria, and original Unit 5 result files were preserved.

**Unit 6, Milestone 3 assistance:** AI prepared the blank staff-label worksheet and checked its format, completeness, and preservation of the original post texts. I supplied all 30 initial labels myself before providing the TF's answer key. After the key arrived, AI verified its IDs and labels, renamed it to `data/staff_labels.csv` without changing its contents, ran `agreement.py`, and helped document the actual comparison and its limits. My initial labels were preserved unchanged. The measured result was 30 matches and zero disagreements, so no disagreement adjudications were needed.

**Unit 6, Milestone 4 assistance:** AI checked each original acceptance target against the saved three-seed results, counted the unchanged dataset, and helped write the verdicts and evidence-based diagnoses. It also used the cached DistilBERT tokenizer for a read-only input-length check. This analysis did not run a model, change training settings, or alter the dataset, targets, or saved results. Possible underlying causes are identified as hypotheses rather than established findings.


<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

The zero-shot baseline is `facebook/bart-large-mnli`, using the three definitions in [label_definitions.txt](label_definitions.txt) without fine-tuning on my dataset. [baseline_results.json](baseline_results.json) confirms that it used those definitions and the held-out set. Its 30 saved prediction records match the text and labels in [test_split.csv](test_split.csv), in the same order. The trained column comes from the original seed-42 DistilBERT run in [results.json](results.json).

Difference means **Trained minus Baseline**, calculated before rounding. Positive values favor the trained model; negative values favor the baseline.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.533 | 0.467 | -0.067 |
| Macro F1 | 0.522 | 0.212 | -0.310 |
| F1 — `computation` | 0.545 | 0.000 | -0.545 |
| F1 — `conceptual` | 0.667 | 0.000 | -0.667 |
| F1 — `proof_help` | 0.353 | 0.636 | +0.283 |

**What I predicted before I looked:**

> Before running the baseline, I expected the fine-tuned model to perform better because it was trained on mathematics questions from this community.

This is my pre-run note, added to the README after completing the baseline run.

**What the gap actually means:**

The outcome went against my prediction: the baseline correctly classified 16 of 30 questions, compared with 14 of 30 for my trained model. The baseline's accuracy was about 6.7 percentage points higher, and its macro F1 was higher by approximately 0.310. On this held-out set, my fine-tuned model performed worse overall than the general zero-shot model.

The trained model had a higher `proof_help` F1, but its saved confusion counts show that it predicted `proof_help` for every test question. That gave it recall of 1.000 for that label, with precision of 0.467, while both other labels had F1 of 0.000. The baseline correctly classified all six `computation` questions and seven of ten `conceptual` questions. This comparison shows a weakness in the trained model's coverage of the taxonomy. It is one 30-question comparison; the next milestone will test the trained model across three seeds before I make broader claims or choose an improvement.

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Section 6 completed three independent training runs with seeds **42, 7, and 2024**, using `RUN_LABEL = "before"`. The saved results are in [results_three_seeds_before.json](results_three_seeds_before.json), and the execution outputs are in [takemeter.ipynb](takemeter.ipynb). All three runs used **Apple Silicon GPU (MPS), PyTorch 2.14.1**, `distilbert-base-uncased`, 3 epochs, learning rate `2e-5`, batch size 16, and maximum length 128 tokens.

The original 200-question dataset and acceptance targets are unchanged. The temporary Section 4/5 switches stayed `False`, so this experiment did not overwrite `results.json` or `test_split.csv`. Section 6 now releases completed models between seeds to avoid retaining the previous model while allocating the next one; the training and evaluation procedures are unchanged.

The measurements below use the five original criteria. Criteria 1, 2, 3, and 5 must pass for every seed; Criterion 4 checks the full dataset once. Verdicts use the unrounded saved values, with the `1e-12` tolerance specified for Criterion 5's confidence gap. The explanations incorporate the completed staff-label agreement check.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 | 0.467 | 0.467 | 0.467 | MISSED |
| 2. Macro F1 | ≥ 0.65 | 0.212 | 0.212 | 0.212 | MISSED |
| 3. Every label's F1 | Each ≥ 0.60 | Minimum 0.000 | Minimum 0.000 | Minimum 0.000 | MISSED |
| 4. Dataset size and balance | ≥ 200 reviewed questions; each label 20%–60% | 200; shares 20%–47% | 200; shares 20%–47% | 200; shares 20%–47% | MET |
| 5. Prediction confidence | Most-confident third accuracy ≥ 0.80; gap over least-confident third ≥ 0.10 | High 0.700; gap 0.200 | High 0.700; gap 0.500 | High 0.900; gap 0.800 | MISSED |

Criterion 3 reports the minimum label F1 above; the individual values are below. Criterion 4 checks the same full dataset for all seeds: 94 `proof_help` (47%), 40 `computation` (20%), and 66 `conceptual` (33%). Repeating those values does not represent three separate dataset reviews. For Criterion 5, each confidence third contains 10 predictions. Least-confident-third accuracy was 0.500, 0.200, and 0.100 respectively; gaps are calculated before rounding.

**Per-label performance and spread:**

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.467 | 0.467 | 0.467 | 0.000 |
| Macro F1 | 0.212 | 0.212 | 0.212 | 0.000 |
| F1 — `proof_help` | 0.636 | 0.636 | 0.636 | 0.000 |
| F1 — `computation` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `conceptual` | 0.000 | 0.000 | 0.000 | 0.000 |

Each seed produced 139 training, 31 validation, and 30 test examples. Every test set contained 14 `proof_help`, 6 `computation`, and 10 `conceptual` questions. The six-example `computation` support remains small.

Accuracy ranged from 0.467 to 0.467, a spread of 0.000. All three runs predicted `proof_help` for every test question. The identical class counts and predictions explain the identical accuracy and F1 values across different splits. This is a consistent majority-label prediction pattern; the zero spread does not establish that the classifier learned all three assistance types.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Section 7 prints the first run's matrix, **seed 42**. The other two seeds produced the same counts. This table describes one 30-question test set; it does not pool the three test sets.

| true \ predicted | proof_help | computation | conceptual |
|---|---|---|---|
| **proof_help** | 14 | 0 | 0 |
| **computation** | 6 | 0 | 0 |
| **conceptual** | 10 | 0 | 0 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

The largest off-diagonal count is **10 real `conceptual` questions predicted as `proof_help`**, compared with **0 real `proof_help` questions predicted as `conceptual`**. The model also classified all six `computation` questions as `proof_help`. These results establish the direction of the errors. The diagnoses below distinguish this observed behavior from possible underlying causes and explain the limits of the staff agreement evidence.


---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.70 for every seed | MISSED | Every seed classified 14/30 questions correctly: 0.466667, below 0.70. |
| 2 | Macro F1 | ≥ 0.65 for every seed | MISSED | Every seed scored 0.212121, below 0.65. |
| 3 | F1 for every label | Each label ≥ 0.60 for every seed | MISSED | `proof_help` scored 0.636364, but `computation` and `conceptual` both scored 0.000 for all three seeds. |
| 4 | Dataset size and balance | ≥ 200 reviewed questions; every label 20%–60% | MET | The unchanged dataset has 200 reviewed records: 94 `proof_help` (47%), 40 `computation` (20%), and 66 `conceptual` (33%). Counts, notes, and the recorded review confirmations support this check. |
| 5 | Prediction confidence | Most-confident third accuracy ≥ 0.80 and gap ≥ 0.10 for every seed | MISSED | Most-confident-third accuracy was 0.700, 0.700, and 0.900; the gaps were 0.200, 0.500, and 0.800. All gaps passed, but seeds 42 and 7 missed the accuracy threshold. Seed 2024 passing alone is insufficient. |

**Diagnoses**

**Criterion 1 — accuracy:** The immediate cause of the miss is majority-label prediction. Each matrix in [results_three_seeds_before.json](results_three_seeds_before.json) shows all 30 predictions going to `proof_help`. This gets the 14 true proof questions right and all 16 other questions wrong, fixing accuracy at 14/30 regardless of which individual posts the split contains. Repeating this pattern across all three seeds supports a persistent failure to distinguish assistance types, rather than a single unfavorable split.

**Criterion 2 — macro F1:** The same prediction pattern leaves two labels with no successful predictions. Macro F1 gives all three labels equal weight, so `(0.636364 + 0 + 0) / 3` is approximately 0.212121. The model's performance on the majority label cannot compensate for its failure on `computation` and `conceptual`. This directly explains the miss without assuming a particular cause for the failure to learn those distinctions.

**Criterion 3 — per-label F1:** Every seed misses all six `computation` and all ten `conceptual` test questions. Both labels have precision, recall, and F1 of zero. `proof_help` achieves recall of 1.000 because every question receives that label, but its precision is only 14/30. The observed problem affects both non-majority labels, beyond the hardest `proof_help`/`conceptual` boundary. `computation` has only 28 training and six test examples, which limits learning evidence and makes its test metrics sensitive to individual cases; however, small test support alone does not explain the complete absence of predictions for both labels across all seeds.

**Criterion 5 — confidence:** The confidence ranking does not consistently achieve the required reliability. For seeds 42 and 7, seven of the ten most-confident predictions were correct; the target requires at least eight. Their gaps over the least-confident third pass, but a relative improvement cannot replace the separate 0.80 accuracy requirement. For seed 42, mean winning confidence is approximately 0.511851 on correct predictions and 0.511015 on incorrect ones, showing very little separation. Seed 2024 reaches nine correct predictions in its most-confident third, but that ranking outcome does not recover either of the unpredicted labels or satisfy the across-seed requirement. Each group contains only ten predictions, so one changed outcome moves its accuracy by ten percentage points; the original target still applies.

**Evidence about possible underlying causes:** A read-only check with the cached `distilbert-base-uncased` tokenizer found that **192/200 posts (96%) exceed the current 128-token limit**: 93/94 `proof_help`, 39/40 `computation`, and 60/66 `conceptual` posts. The CSV retains each complete post, but the training tokenizer truncates the model's input. For example, record 122's closing request, "Can $\eqref{1}$ be proved for $x,y,z>0$?", begins at token 329 in a 353-token post, counting special tokens. That explicit request is beyond the current input window. This establishes loss of potentially relevant text across the corpus; it does not establish that truncation caused every error. A longer input window is a plausible hypothesis for a later controlled experiment.

The completed runs also contain only 27 training updates each. Validation loss decreases over the three epochs for every seed, while validation accuracy stays at 15/31, approximately 48.4%. This is consistent with the model adjusting its scores without learning useful decision boundaries under the current setup. Insufficient training exposure is another possible explanation, but the saved runs do not establish that more epochs would improve held-out performance. Neither hypothesis has been tested as an improvement here.

**What the agreement report establishes:** [agreement_results.json](agreement_results.json) records 30/30 matches and zero disagreements, including all seven staff-marked borderline cases. There is no observed staff disagreement that supports a diagnosis of loosely applied rules in that exercise. However, those posts use the running-forum taxonomy, not the mathematics taxonomy. This evidence neither proves that the 200 math labels are consistent nor identifies a math boundary that should be relabeled. I therefore cannot attribute the model's collapse to annotation inconsistency from this report, and I cannot rule out math-specific label ambiguity either.

**Limits of the diagnosis:** Majority-label prediction, truncated inputs, small per-label samples, and weak confidence separation are observed facts. Their causal contributions to the model's behavior remain uncertain. Criterion 4 passes, so the dataset does not violate the stated balance requirement; passing that check does not guarantee adequate examples for learning. Milestone 5 will test one justified change while keeping the other conditions fixed, rather than treating an untested explanation as established or lowering the original targets.

---

## Agreement Report

**Status:** All 30 initial annotations and the official staff comparison are complete. There were no disagreements to adjudicate.

I labeled all 30 running-forum posts in [my_staff_labels.csv](my_staff_labels.csv) using the definitions and boundary rules in [data/staff_taxonomy.md](data/staff_taxonomy.md). These use the staff categories, separate from my mathematics classifier's taxonomy. Validation confirmed exactly `text,label` columns, 30 valid labels, no duplicate posts, and every original post text preserved unchanged.

| My initial label | Count |
|---|---:|
| `analysis` | 10 |
| `hot_take` | 10 |
| `reaction` | 10 |
| **Total** | **30** |

**Agreement rate: 30 / 30 = 100%.**

The TF supplied the official answer key as `(SEND AFTER THEY ARE DONE) data-staff_labels.csv`. It is saved, with its contents unchanged, as [data/staff_labels.csv](data/staff_labels.csv), the filename expected by the script. All 30 staff IDs match [data/staff_posts.csv](data/staff_posts.csv), with no duplicate or missing IDs and only the three allowed labels.

Running `python agreement.py` produced [agreement_results.json](agreement_results.json): 30 posts compared, 30 agreements, 0 disagreements, and 0 staff posts missing from my annotations. My initial annotation file was preserved byte for byte during the comparison.

**Disagreements and adjudication:** None. Every initial label matched the official key, so there are no disagreement entries or final calls to record. This reports the actual comparison; no initial labels were changed to match staff.

**What the pattern in my disagreements tells me:**

There is no disagreement pattern in this set. The staff key marks seven posts as borderline, so the perfect agreement should not be interpreted as evidence that every post was obvious. The written rules provide concrete distinctions for those cases: a fact must carry the argument, a general claim differs from a personal response, and recency alone does not determine a label.

For example, the key labels `sp_005` as `analysis` because its numerical comparisons support the argument, while `sp_014` is `hot_take` because its statistic decorates an unsupported conclusion. Both matched my initial labels. These examples illustrate the evidence-versus-keywords distinction that the staff rules require.

The result shows that my initial decisions aligned with staff on this 30-post exercise. It does not prove that every boundary is unambiguous or that my separate 200-question mathematics dataset is error-free. The staff exercise uses a different community and taxonomy, so it cannot by itself establish the cause of the mathematics model's tendency to predict `proof_help` for every question. That limitation will remain explicit in the diagnoses.

---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
