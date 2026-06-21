# Data Collection Guide — r/nba

Target: 225+ posts total (~75 per label). All content is public.

---

## Where to Find Each Label

### `analysis` (~75 posts)
Best sources:
- r/nba posts with the "Analysis" or "Discussion" flair
- Long-form comments in post-game threads that cite stats
- Weekly discussion megathreads (e.g., "Film Room Friday", "Stat of the Day")
- Posts referencing Basketball-Reference, Cleaning the Glass, or PBP stats
- Search r/nba for: "according to stats", "per 36", "TS%", "defensive rating", "win shares"

Signs it's analysis: multiple sentences, specific numbers, logical flow from evidence to conclusion.

### `hot_take` (~75 posts)
Best sources:
- r/nba front page — provocative titles get upvoted
- Posts starting with "[Player] is overrated/underrated/washed/the GOAT"
- Comments in GOAT debates (Jordan vs. LeBron threads are goldmines)
- "Unpopular opinion:" posts (though check — some are actually analysis)
- Search r/nba for: "overrated", "unpopular opinion", "change my mind", "always will be"

Signs it's a hot take: short, declarative, no evidence, often inflammatory framing.

### `reaction` (~75 posts)
Best sources:
- Game threads (posted during or right after games) — huge source
- Post-game threads in the first hour after final buzzer
- Posts about breaking news: trades, injuries, signings
- Comments with ALL CAPS, excessive punctuation, or pure emotional language
- Search r/nba for: recent game threads, "I can't believe", "what just happened", "season is over"

Signs it's a reaction: event-triggered, emotional language, no argument being made.

---

## Collection Workflow

1. Open Reddit and go to r/nba
2. Browse the sources above for your current target label
3. Copy the post title + body text (or top-level comment) into your spreadsheet
4. Paste into the `text` column, add the `label`, leave `notes` blank unless it was a hard call
5. For hard calls: write a brief note in the `notes` column explaining what you decided and why
6. Every ~30 posts, check your label distribution — adjust collection targets if needed

---

## Spreadsheet Setup

Open `data/dataset.csv` in Excel or Google Sheets. Columns:
- **text**: the post or comment (trim to ~300 words max if very long)
- **label**: one of `analysis`, `hot_take`, `reaction`
- **notes**: optional — use for any example you found difficult to label

---

## Pre-Labeling with AI (optional speedup)

1. Collect 30–50 posts without labeling them first
2. Paste them (numbered) into the prompt in `data/prelabeling_prompt.md`
3. Run it through Claude or Groq
4. Review and correct every label before adding to your CSV
5. Track how many you corrected — disclose this in your README

---

## Label Balance Check

After every 50 posts, count per label. Target range per label: 65–85.
If any label exceeds 85 posts before others reach 65, stop collecting that label and focus on the others.

---

## Knicks Context

Right now r/nba is especially active with Knicks content. This is a great opportunity to collect:
- **reaction** posts from game threads and the championship celebration
- **hot_take** posts about the Knicks' dynasty potential, Jalen Brunson's ranking, etc.
- **analysis** posts breaking down what made this Knicks team special

Lean into this moment — the data will be richer for it.
