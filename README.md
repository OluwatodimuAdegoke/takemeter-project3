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
| `analysis` | 0.59 | 0.83 | 0.69 |
| `hot_take` | 0.33 | 0.27 | 0.30 |
| `reaction` | 0.88 | 0.64 | 0.74 |
| **macro avg** | **0.60** | **0.58** | **0.58** |

### Zero-shot Baseline — Per-Class Metrics

| Label | Precision | Recall | F1 |
|-------|-----------|--------|-----|
| `analysis` | 0.82 | 0.75 | 0.78 |
| `hot_take` | 0.73 | 0.73 | 0.73 |
| `reaction` | 0.75 | 0.82 | 0.78 |
| **macro avg** | **0.77** | **0.77** | **0.76** |

---

## Analysis

### Why the Fine-tuned Model Underperformed

The regression is real and consistent across multiple training runs. Three factors explain it:

**1. The `hot_take` label is the entire gap.**
The fine-tuned model achieves F1 of 0.30 on `hot_take` vs. 0.73 for the Groq baseline. `analysis` (0.69) and `reaction` (0.74) are both competitive. The `hot_take`/`analysis` boundary requires *pragmatic* judgment: does this person assert a conclusion, or reason toward one? Surface-level token patterns — player names, game references, specific stats — appear in both classes. DistilBERT cannot reliably distinguish "mentioning something specific" from "reasoning from something specific."

**2. Groq has a structural pre-training advantage on this task.**
llama-3.3-70b-versatile was pre-trained on billions of tokens of internet text including sports discourse, where "hot take" is a culturally loaded term with well-understood meaning. The zero-shot system prompt activates knowledge the model already has. DistilBERT has no such prior — it must learn the boundary from 75 labeled training examples, which is insufficient for a nuanced pragmatic distinction.

**3. 75 examples per class is near the lower bound for fine-tuning.**
BERT-family models typically require 500–1,000 labeled examples per class to reliably learn soft semantic boundaries. With 75, the model learned `analysis` and `reaction` reasonably well (both are more syntactically distinct), but could not generalize the `hot_take` boundary.

### What Would Flip the Result

With ~500 labeled examples per class (1,500 total), fine-tuned DistilBERT would likely surpass 80% accuracy and outperform the zero-shot baseline. Small models fine-tuned on sufficient in-domain data consistently outperform large zero-shot models on narrow classification tasks. This project fell short of that data threshold.

### What the Model Did Learn

Despite the overall regression, the fine-tuned model is not random. It achieves near-perfect precision on `reaction` (0.88) and solid recall on `analysis` (0.83), demonstrating it learned meaningful signal about both of those classes. The failure is specifically localized to the `hot_take`/`analysis` confusion — a boundary that even human annotators find difficult.

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
