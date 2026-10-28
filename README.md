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

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



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
