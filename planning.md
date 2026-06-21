# TakeMeter — Planning Document

## Community

**Chosen community:** r/nba (Reddit)

r/nba is one of the largest and most active sports communities on the internet, with millions of members discussing NBA games, players, trades, and history daily. The discourse is unusually varied in quality: the same subreddit hosts rigorous statistical breakdowns, gut-reaction hot takes, emotional postgame venting, and open debate threads — sometimes in the same comment section. This variance makes it an excellent fit for a discourse quality classifier.

A personal motivating factor: I'm based in New York, and the Knicks just won — making the NBA an especially live and emotionally charged topic right now. The subreddit is flooded with posts ranging from euphoric reactions to serious championship-window analysis, making this an ideal moment to collect a diverse, high-energy dataset that captures the full range of discourse types.

The distinctions we're measuring (analysis vs. hot take vs. reaction) are ones that r/nba users themselves invoke constantly — phrases like "that's just a hot take," "do you have any data for that?", and "this is actually a good analysis" are native to the community. The task is grounded in real community norms, not imposed from outside.

---

## Label Taxonomy

### Label 1: `analysis`
**Definition:** The post makes a structured argument backed by statistics, historical comparison, or tactical observation. The evidence is specific and verifiable, and the post reasons from that evidence rather than just citing it for effect.

**Example 1:**
> "People sleep on Draymond's defensive impact. Over the last 3 seasons, the Warriors' defensive rating is 108.4 with him on the floor vs. 114.7 without — a gap larger than any other player on the team. He's irreplaceable in that system."

*Why it's analysis:* Specific, verifiable stats. The post argues from the numbers rather than just asserting.

**Example 2:**
> "The Celtics' halfcourt offense ranks 4th in the league at 1.12 PPP, but they drop to 18th in transition. That's why they struggle in fast-paced playoff series — they can't generate easy buckets when teams push pace against them."

*Why it's analysis:* Makes a tactical observation, supports it with ranked statistics, and draws a specific conclusion.

---

### Label 2: `hot_take`
**Definition:** A bold, confident opinion stated without supporting evidence. The claim may be true, but the post asserts rather than argues. The framing is typically provocative or contrarian.

**Example 1:**
> "LeBron is overrated. Plain and simple. Jordan would have 9 rings if he played in this era."

*Why it's a hot take:* Sweeping claims, no evidence, maximum confidence.

**Example 2:**
> "Trae Young will never win a championship. He's a regular-season stat-padder and always will be."

*Why it's a hot take:* Declarative, no supporting reasoning, designed to provoke.

---

### Label 3: `reaction`
**Definition:** An immediate emotional response to a specific event or moment — a game, a play, a trade announcement, an injury update. The post is expressing a feeling in real time. Little to no argument; the point is to share the emotion.

**Example 1:**
> "LUKA JUST HIT THAT FROM HALFCOURT ARE YOU KIDDING ME"

*Why it's a reaction:* Pure emotion, specific moment, no argument.

**Example 2:**
> "I genuinely can't believe they traded him. This is the worst day of my fan life. Season is cooked."

*Why it's a reaction:* Emotional response to a specific event (trade), no analysis of whether the trade was good or bad strategically.

---

## Hard Edge Cases

### Edge Case 1: The stat-drop that's really a hot take
**Post:**
> "LeBron's playoff win rate against top-seeded opponents is below .500 — he's overrated."

**Which labels could apply:** `analysis` (cites a specific stat) or `hot_take` (accusatory framing, one cherry-picked number).

**Decision rule:** If the evidence is specific and verifiable AND the post reasons from it as part of a broader argument, label it `analysis`. If the stat exists only to sound credible while the post is really just asserting a conclusion, label it `hot_take`. Cherry-picked single stats used as ammunition (not as part of a structured argument) → `hot_take`.

---

### Edge Case 3: The emotional post about a trade or offseason move
**Post:**
> "I can't believe they signed him to a max deal. This franchise is a disaster."

**Which labels could apply:** `reaction` (emotional, event-triggered) or `hot_take` (makes a claim about the franchise).

**Decision rule:** If the post's primary purpose is expressing emotion about what just happened — and the "claim" is more venting than arguing — label it `reaction`. If the post leads with an opinion and makes a case for it (even a weak one), label it `hot_take`. The post above is `reaction`.

---

## Data Collection Plan

**Source:** r/nba on Reddit. Posts and top-level comments from the main subreddit feed, including game threads, discussion posts, and regular posts. All content is public.

**Target volume:** 225 examples minimum (buffer above the 200 required), split evenly across labels:
- `analysis`: ~75 examples
- `hot_take`: ~75 examples
- `reaction`: ~75 examples

**Collection method:** Manual collection by reading and copy-pasting into a spreadsheet. This keeps annotation close to the actual text and avoids scraping overhead for a 200-example dataset.

**If a label is underrepresented:** If any label falls below 60 examples after the initial collection pass, collect additional posts specifically targeting that label before finalizing the dataset. Game threads are a reliable source for `reaction` examples; analytical posts and megathreads for `analysis`; r/nba front page for `hot_take`.

**CSV format:**
```
text, label, notes
"post text here", hot_take, "difficult — could be analysis, decided hot_take because..."
```

**Train/val/test split:** 70% / 15% / 15% (handled automatically by the Colab notebook).

**Label distribution constraint:** No single label should exceed 35% of the total dataset. If any label exceeds this after collection, balance by collecting more examples from underrepresented labels.

---

## Evaluation Metrics

**Primary metric: per-class F1 score**

Accuracy alone is misleading here because the task has 4 classes. A model that predicts `hot_take` 60% of the time could achieve decent accuracy if hot takes are the most common label — but it would be useless for classifying the other labels. Per-class F1 (harmonic mean of precision and recall per label) catches this and tells us which distinctions the model has and hasn't learned.

**Secondary metric: overall accuracy**

Used for the baseline comparison — gives a single number to compare fine-tuned vs. zero-shot performance at a glance.

**Confusion matrix**

Essential for identifying which label pairs the model confuses and in which direction. A high off-diagonal count at (true=`analysis`, predicted=`hot_take`) tells us the model can't distinguish a stat-backed argument from an assertive claim — a specific, diagnosable failure.

**Why not just accuracy:**
With 3 classes and ~33% expected random baseline, accuracy alone doesn't reveal whether the model learned the hard boundaries (e.g., `analysis` vs. `hot_take`) or just the easy ones (e.g., `reaction` vs. `analysis`).

---

## Definition of Success

A classifier is **genuinely useful** for deployment in a community tool (e.g., a post flair suggester or discourse quality tracker) if:

- Overall accuracy ≥ 70% on the test set
- Per-class F1 ≥ 0.60 for all three labels (no label is completely unlearned)
- The fine-tuned model outperforms the zero-shot baseline by at least 10 percentage points in overall accuracy

**Minimum acceptable for this project:**
- Fine-tuned model accuracy > zero-shot baseline (fine-tuning must add value)
- At least 2 of 3 labels achieve F1 ≥ 0.65

If fine-tuned performance is below baseline across the board, that's a signal to investigate label leakage, class imbalance, or annotation inconsistency — not just to report the number.

---

## AI Tool Plan

### 1. Label stress-testing
Before annotating 200 examples, I will give Claude my label definitions and edge case descriptions and ask it to generate 10–15 posts that sit at the boundary between two labels — specifically `analysis` vs. `hot_take` and `reaction` vs. `hot_take`. If it produces posts I can't classify cleanly, the definitions need tightening. I will revise the definitions before beginning annotation.

### 2. Annotation assistance
I will use an LLM (Claude or Groq) to pre-label a batch of ~100 examples using my label definitions. I will then review and correct every pre-assigned label myself. I will not use pre-labels as a rubber stamp — each one gets read and verified. In my README's AI usage section, I will disclose which examples were pre-labeled and what percentage I corrected.

### 3. Failure analysis
After running evaluation, I will paste my misclassified examples into Claude and ask it to identify common patterns — e.g., "do these tend to be short posts?", "do they share specific framing like sarcasm or rhetorical questions?", "is there a consistent pair of labels being confused?" I will then verify those patterns myself by re-reading the examples, and note any patterns I had to correct or discard from the AI's output.

---

## Stretch Features (planned additions)

*This section will be updated before starting any stretch feature.*

- [ ] Inter-annotator reliability (Cohen's kappa with at least one other annotator on 30+ examples)
- [ ] Confidence calibration analysis
- [ ] Error pattern analysis (systematic, not just individual)
- [ ] Deployed interface (Gradio or similar, accepting new posts and returning label + confidence)
