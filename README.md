# TakeMeter

A fine-tuned text classifier that categorizes r/nba Reddit posts by discourse quality — distinguishing structured analysis from hot takes and emotional reactions.

---

## Community

**r/nba** (Reddit) — one of the largest sports communities on the internet, with millions of members discussing NBA games, players, trades, and history daily. The subreddit is unusually varied in discourse quality: the same thread can contain rigorous statistical breakdowns, gut-reaction hot takes, and pure emotional venting. This variance makes it an ideal testbed for a discourse quality classifier.

The distinctions TakeMeter measures (`analysis`, `hot_take`, `reaction`) are ones r/nba users invoke constantly — phrases like "that's just a hot take," "do you have any data for that?", and "this is actually a good analysis" are native to the community. The classifier is grounded in real community norms, not imposed from outside.

---

## Label Taxonomy

| Label | Definition |
|-------|-----------|
| `analysis` | Makes a structured argument backed by statistics, historical comparison, or tactical observation. Evidence is specific and verifiable; the post reasons *from* that evidence to a conclusion. |
| `hot_take` | A bold, confident opinion stated without supporting evidence. Asserts rather than argues. Often provocative or contrarian. A single stat used to *sound* credible (but not reasoned from) is still a hot_take. |
| `reaction` | An immediate emotional response to a specific event — a game, play, trade, or injury. Expresses a feeling in real time with little to no argument. |

### Hard Edge Case: `analysis` vs. `hot_take`

The trickiest boundary in the taxonomy. Decision rule: if the post cites specific, verifiable evidence **and reasons from it** to a conclusion, it's `analysis`. If the post drops a stat for rhetorical effect but the conclusion doesn't actually follow from the evidence, or if no evidence is offered at all, it's `hot_take`.

---

## Dataset

| Split | Size |
|-------|------|
| Train | ~158 examples (70%) |
| Validation | ~34 examples (15%) |
| Test | 34 examples (15%) |
| **Total** | **226 examples** |

**Label distribution:**

| Label | Count |
|-------|-------|
| `analysis` | 75 |
| `hot_take` | 75 |
| `reaction` | 76 |

### Data Collection

All examples were collected manually from public r/nba posts and comment threads. Sources included post-game threads, trade discussion threads, megathreads about player performance, and championship parade coverage. Each example was read and labeled by a single annotator (the author).

**Threads used (sample):**
- JR Smith 2018 Finals blunder discussion
- Giannis "no failure in sports" speech
- Knicks/Spurs Finals stat analysis threads
- SGA Jordan-esque efficiency debate
- Steven Adams boxing out commentary
- Zaccharie Risacher frustrating rookie season
- Knicks championship parade coverage

**Annotation approach:** Approximately 40% of examples were pre-labeled using an LLM (Claude/Groq) with a structured prompt defining each label and providing decision rules. Every pre-assigned label was reviewed and corrected by the author before inclusion. Pre-labeling was used only as a first-pass suggestion, not as a ground truth.

---

## Model

**Architecture:** `distilbert-base-uncased` fine-tuned for sequence classification

**Training hyperparameters:**
- Epochs: 5
- Learning rate: 2e-5
- Batch size: 16
- Warmup steps: 50
- Weight decay: 0.01

**Framework:** HuggingFace Transformers + Trainer API, trained on Google Colab T4 GPU (~3 minutes).

---

## Evaluation Results

### Accuracy Comparison

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | **73.5%** |
| Fine-tuned DistilBERT | 61.8% |

**Result: fine-tuning regression of 11.8 percentage points.**

### Fine-tuned Model — Per-Class Metrics

| Label | Precision | Recall | F1 |
|-------|-----------|--------|-----|
| `analysis` | 0.53 | 0.83 | 0.64 |
| `hot_take` | 0.00 | 0.00 | 0.00 |
| `reaction` | 0.73 | 1.00 | 0.84 |
| **macro avg** | **0.42** | **0.61** | **0.49** |

### Zero-shot Baseline — Per-Class Metrics

| Label | Precision | Recall | F1 |
|-------|-----------|--------|-----|
| `analysis` | 0.82 | 0.75 | 0.78 |
| `hot_take` | 0.73 | 0.73 | 0.73 |
| `reaction` | 0.75 | 0.82 | 0.78 |
| **macro avg** | **0.77** | **0.77** | **0.76** |

### Confusion Matrix — Fine-tuned Model

Rows = true label, columns = predicted label. Test set: 34 examples.

|                  | pred: `analysis` | pred: `hot_take` | pred: `reaction` |
|------------------|:----------------:|:----------------:|:----------------:|
| true: `analysis` |        10        |        0         |        2         |
| true: `hot_take` |        9         |        0         |        2         |
| true: `reaction` |        0         |        0         |        11        |

The most striking result: **the model never predicts `hot_take` at all.** Every one of the 11 true hot_take examples was predicted as either `analysis` (9) or `reaction` (2). The `hot_take` column is entirely zero. The model learned `reaction` almost perfectly (11/11), learned `analysis` reasonably (10/12), and effectively abandoned `hot_take` as a distinct class.

---

## Error Analysis

### Which labels are being confused

The confusion matrix reveals something more severe than a blurry boundary: **the model stopped predicting `hot_take` entirely.** Zero examples were predicted as `hot_take` across the full test set. All 11 true hot_takes were absorbed into `analysis` (9) or `reaction` (2). Meanwhile, `reaction` was predicted perfectly — 11/11 correct, zero false positives. The model effectively collapsed to a two-class system: everything is either `analysis` or `reaction`, and `hot_take` ceased to exist as an output.

### Why this boundary is hard

The `hot_take`/`analysis` boundary is pragmatic, not syntactic. Both labels can reference specific players, cite game situations, and use confident declarative sentences. The difference is whether the evidence is *reasoned from* or just *deployed for effect*. "His playoff win rate is below .500 — he's overrated" and "Over the last 3 seasons, his win rate in closeout games is 78% — he thrives under pressure" have nearly identical surface structure: stat + claim. The distinction requires understanding whether the conclusion actually follows from the evidence, which is a form of logical reasoning DistilBERT cannot reliably perform from 75 training examples.

### Specific wrong predictions

**Error 1: `hot_take` predicted as `analysis`**

> "prime Jordan would had beat the spurs in game 7 and beat the Knicks in 6 games. No lies."

True label: `hot_take` — Predicted: `analysis`

This post makes a specific counterfactual claim about two series outcomes, which superficially resembles the kind of evidence-grounded reasoning in `analysis` posts. The model has learned to associate specificity (named teams, specific series, concrete claims) with `analysis`. But the post offers no reasoning — it just asserts the outcome with "no lies" as its only justification. The model read the specificity of the claim as a signal of analysis rather than reading the absence of any supporting argument. This is the core failure: specificity ≠ reasoning.

**Error 2: `hot_take` predicted as `analysis`**

> "LeBron is overrated. Plain and simple. Jordan would have 9 rings if he played in this era."

True label: `hot_take` — Predicted: `analysis` (confidence: 39.4%)

The LeBron/Jordan GOAT debate is one of the most common framing patterns in r/nba `analysis` posts — the model has seen many examples that start with "LeBron" and "Jordan" in an argumentative context and learned to associate that framing with `analysis`. Here the model is being fooled by topic, not structure. The post contains zero evidence; it's a pure assertion. But because the subject matter (GOAT debate) overlaps so heavily with analytical threads in the training data, the model misread the topic as the label signal. This is a data distribution problem: the training set likely has more `analysis` than `hot_take` examples about LeBron/Jordan comparisons, so the model learned the wrong prior.

**Error 3: `analysis` predicted as `reaction`**

> "People sleep on Draymond's defensive impact. Over the last 3 seasons, the Warriors' defensive rating is 108.4 with him on the floor vs. 114.7 without — a gap larger than any other player on the team. He's irreplaceable in that system."

True label: `analysis` — Predicted: `reaction` (confidence: 40.6%)

This is the most surprising error. The post contains specific, verifiable on/off statistics and reasons from them to a clear conclusion. By the label definition it is unambiguously `analysis`. The model predicted `reaction` at only 40.6% confidence — barely above random (33%) — which means it was genuinely uncertain. One likely explanation: the opening phrase "People sleep on…" is a common emotional/reactive framing device on Reddit, and the model may have latched onto that signal. The rest of the post is analytical, but if the model weights early tokens heavily, the "people sleep on" opener could have pushed the prediction toward `reaction`. This suggests the model is partly encoding discourse framing cues rather than the structural argument within the post.

### Is this a labeling problem or a data problem

The labels on the wrong examples above are clear and consistently applied — there is no annotation inconsistency driving these errors. The problem is in the training data distribution. With only 75 `hot_take` examples, the model did not see enough variation in how hot takes can be framed (specific counterfactuals, GOAT debates, stat-drops) to learn that framing ≠ reasoning. The `hot_take`/`analysis` boundary requires examples that explicitly show the contrast at the sentence level, not just at the topic level. The current dataset likely has topic-level confounds (LeBron/Jordan appears in both classes) that the model resolved by overfitting to topic rather than structure.

### What would fix it

When a fine-tuned model stops predicting a class entirely, it usually means the class boundary was too weak to survive optimization — the model learned it was "safer" (lower loss) to merge `hot_take` into the neighboring classes than to commit to a noisy boundary. Two things would help: (1) roughly 400–500 labeled `hot_take` examples to give the class enough weight in the loss landscape, and (2) examples that explicitly contrast a hot_take and an analysis on the same topic — showing the model side-by-side what the structural difference looks like when topic is held constant.

### Sample Classifications

| Post (truncated) | True Label | Predicted | Confidence |
|------------------|-----------|-----------|------------|
| "People sleep on Draymond's defensive impact. Over the last 3 seasons, the Warriors' defensive rating is 108.4 with him on the floor vs. 114.7 without…" | `analysis` | `reaction` ✗ | 40.6% |
| "LeBron is overrated. Plain and simple. Jordan would have 9 rings if he played in this era." | `hot_take` | `analysis` ✗ | 39.4% |
| "I genuinely can't believe they traded him. This is the worst day of my fan life. Season is cooked." | `reaction` | `reaction` ✓ | — |
| "The Celtics' halfcourt offense ranks 4th in the league at 1.12 PPP, but they drop to 18th in transition. That's why they struggle in fast-paced playoff series." | `analysis` | `analysis` ✓ | — |

The `reaction` prediction on the third example is a case where the model is on solid footing: pure emotional language ("can't believe", "worst day", "season is cooked") with no argument is a clean signal for `reaction`, and the model has learned this pattern reliably (precision 0.88). The fourth example is correctly identified as `analysis` — multiple ranked statistics reasoned to a conclusion is the clearest possible signal for that class.

---

## Reflection: What the Model Captured vs. What Was Intended

The label definitions describe a *pragmatic* distinction — how a post uses evidence, not just whether it contains evidence. The fine-tuned model learned a *surface* approximation of that distinction.

For `reaction`, the approximation holds well. Emotional language, capitalization, exclamation patterns, and event-reference framing are consistent surface signals for the label. The model captured these reliably.

For `analysis`, the approximation partially holds. The model learned that multi-sentence posts with player names, statistics, and logical connectors ("because", "that's why", "which means") tend to be `analysis`. Recall is high (0.83) — it catches most analysis examples. But precision is only 0.59, meaning it over-triggers on `hot_take` posts that share those surface features.

For `hot_take`, the approximation fails. The intended decision rule was: bold claim + no reasoned argument = `hot_take`. The model would need to understand argument structure — the difference between citing something and reasoning from it — to apply this rule. Instead, the model appears to have learned a weaker proxy: short + declarative + no stats ≈ `hot_take`. This works for the obvious cases (one-sentence declarations with no specifics) but fails when a `hot_take` is phrased with specificity ("Jordan would have 9 rings in this era" names a specific number) or when an `analysis` post opens with reactive framing ("People sleep on…").

The fundamental gap: the label taxonomy was designed to capture *what a post is doing*, while the model learned to recognize *what a post looks like*. For two of the three labels, those happen to correlate well enough. For `hot_take`, the correlation breaks down because the defining feature of a hot take — asserting without arguing — is structural and pragmatic, not lexical.

---

## AI Tool Usage Disclosure

This project used AI assistance in the following ways:

| Task | Tool | How it was used |
|------|------|-----------------|
| Pre-labeling | Claude / Groq | ~40% of examples were pre-labeled using a structured prompt. Every pre-label was reviewed and corrected by the author. |
| Label stress-testing | Claude | Generated boundary cases between `analysis` and `hot_take` to tighten label definitions before annotation began. |
| Failure analysis | Claude | After evaluation, misclassified examples were analyzed to identify systematic patterns (e.g., all `hot_take→analysis` errors referenced specific events). |
| Colab notebook debugging | Claude | Diagnosed training issues including warmup_steps misconfiguration and model checkpoint reuse. |

No AI tool was used to generate dataset examples, write final label assignments without review, or write this README.

---

## How to Run

1. Upload `data/dataset.csv` to Google Colab
2. Open `notebook/takemeter_colab.ipynb`
3. Set your Groq API key in the secrets panel
4. Run all cells in order

Runtime: ~3 minutes on T4 GPU (free tier).
