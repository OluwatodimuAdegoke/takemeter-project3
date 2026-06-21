# Pre-Labeling Prompt (copy this into Claude or Groq)

Use this prompt to pre-label batches of r/nba posts before you review and correct them yourself.

---

## Prompt

You are helping label a dataset of r/nba Reddit posts for a text classification project. Your job is to assign exactly one label to each post using the definitions and decision rules below.

---

### Labels and Definitions

**analysis**
The post makes a structured argument backed by statistics, historical comparison, or tactical observation. The evidence is specific and verifiable, and the post reasons from that evidence. It's not enough to cite one number — the post must use evidence to support a conclusion.

**hot_take**
A bold, confident opinion stated without supporting evidence. The post asserts rather than argues. Framing is often provocative or contrarian. A post that drops a single stat just to sound credible (but doesn't reason from it) is still a hot_take.

**reaction**
An immediate emotional response to a specific event or moment — a game, a play, a trade, an injury. The post is expressing a feeling in real time. Little to no argument; the point is to share the emotion.

---

### Decision Rules for Hard Cases

1. **Single stat used as ammunition, not reasoning** → `hot_take` (e.g., "His playoff win rate is below .500 — he's overrated.")
2. **Emotional post about a trade/signing** → `reaction` if venting; `hot_take` if making a case
3. **Post that cites multiple stats and reasons from them** → `analysis`

---

### Output Format

For each post, respond with ONLY the label on its own line — no explanation, no punctuation, just the label word. One label per post.

Valid labels: `analysis`, `hot_take`, `reaction`

---

### Posts to Label

[PASTE YOUR UNLABELED POSTS HERE — one per line, numbered]

1. [post text]
2. [post text]
3. [post text]
...

---

### Example Output

1. hot_take
2. analysis
3. reaction

---

**After getting results:** Review every label before accepting it. Correct any that don't match the definitions. Pay special attention to `analysis` vs. `hot_take` — the model will sometimes label assertive-but-evidence-sounding posts as `analysis` when they're really `hot_take`.
