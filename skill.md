---
name: curationascraft
description: >
  Applies the "Curation as Craft" manifesto pipeline: consume artifacts/canvases,
  internalize via analysis, edit (water/fire/paper/scissors ops), and give back via
  zine/blog/gallery publish proposals. Use this skill whenever the user mentions:
  curating a zine, remixing artifacts, publishing from a canvas, atelier pipeline,
  studio compilation, curation suggestions, artifact clustering, or any Curation as
  Craft workflow. Trigger even when the request is partial — e.g. "suggest what to
  cut from my canvas" or "help me remix these into a post".
---

## Activation Triggers
- "Curate zine from canvas X"
- "Remix artifacts Y as blog / post / gallery"
- "Curation suggestions for theme Z"
- "What should I cut / keep / publish?"
- "Cluster these artifacts"
- "Run the atelier pipeline on X"

---

## Phase 1 — Consume

Fetch input artifacts. Sources in priority order:
1. Uploaded JSON/CSV file at `/mnt/user-data/uploads/`
2. Canvas data passed inline in the message
3. Web context via `web_search` for theme enrichment: `search_web(["latest {theme} research 2026"])`

If no source available → ask user to provide artifacts JSON or describe canvas.

---

## Phase 2 — Internalize (inline analysis — no external skill required)

Run this Python block via `execute_code`. Replaces canvasanalyzer dependency entirely.

```python
import json
import pandas as pd
from collections import Counter
import re

# Load artifacts — adjust path or replace with inline data
with open('artifacts.json') as f:
    data = json.load(f)

df = pd.DataFrame(data)

# Required columns: id, content, quality_score, op_time
# Optional: tags, author, type

# --- Theme clustering (keyword frequency, no sklearn needed) ---
def extract_keywords(text, top_n=3):
    words = re.findall(r'\b[a-z]{4,}\b', text.lower())
    stopwords = {'this','that','with','from','have','been','will','they','them'}
    filtered = [w for w in words if w not in stopwords]
    return [w for w, _ in Counter(filtered).most_common(top_n)]

df['keywords'] = df['content'].apply(extract_keywords)
df['cluster'] = df['keywords'].apply(lambda kws: kws[0] if kws else 'uncategorized')

# --- Quality tier ---
df['tier'] = pd.cut(
    df['quality_score'],
    bins=[0, 0.4, 0.7, 1.0],
    labels=['low', 'mid', 'high']
)

print(df[['id', 'cluster', 'tier', 'quality_score']].sort_values('quality_score', ascending=False).to_string())
print("\nCluster distribution:", df['cluster'].value_counts().to_dict())
```

Outputs: ranked artifact table, cluster labels, quality tiers.

---

## Phase 3 — Edit (ops)

Apply curation ops based on analysis results:

| Op | Trigger | Action |
|----|---------|--------|
| **Water** | tier == 'low', not duplicate | Flag for update — add note: "needs enrichment" |
| **Fire** | Duplicate cluster + quality_score < 0.5 | Mark for deletion |
| **Paper** | Gap in theme coverage | Generate new artifact via AI to fill cluster |
| **Scissors** | tier == 'high', strong cluster signal | Select for compilation; reorder by relevance |

Select top 5 for zine/blog by: tier == 'high' → sort by quality_score desc → diversity across clusters.

### Key Heuristics (from essay)

Apply these during Edit to override mechanical op decisions when judgment is needed:

**Bottom-up** — Prioritize novel or unexpected perspectives over polished but predictable ones. A mid-tier artifact covering an underrepresented angle outranks a high-tier one duplicating dominant themes. When ranking for Scissors, weight novelty alongside quality_score.

**Care** — Flag low-confidence decisions explicitly. If a cluster label is ambiguous, a quality_score seems anomalous, or an artifact sits on a tier boundary (e.g. score = 0.41), add a note rather than silently applying the op. Surface these for user review before firing or watering.

**Fusion** — Actively seek cross-cluster combinations. The strongest zine/blog proposals draw from 3+ distinct clusters. If the top 5 Scissors selections collapse into 1–2 clusters, expand to mid-tier artifacts from underrepresented clusters to achieve thematic range.

---

## Phase 4 — Give Back

Produce one of:

**Zine/Blog proposal:**
```
Title: [derived from dominant cluster theme]
Order: [artifact ids in curation order]
Tags: [top keywords]
Publish: POST /zines  { keyword: "{cluster}-fusion", artifacts: [...ids] }
Audience: gated | public  (ask user if unclear)
```

**Gallery impact estimate:**
- High-tier artifacts from 3+ clusters → predict broad reach
- Single-cluster, high-quality → predict niche/deep engagement

**Diff/snapshot:**
```
Fired: [ids]
Watered: [ids + note]
Added (Paper): [new artifact summary]
Final compilation: [ids in order]
```

---

## Fallback — No artifacts.json

If no file provided:
1. Ask user to paste artifact list as JSON array with fields: id, content, quality_score
2. Minimum viable fields: id, content (quality_score defaults to 0.5)
3. Offer to run analysis on description alone if structured data unavailable

---

## Output Table Format

| Step        | Op                         | Artifacts     | Notes                   |
| ----------- | -------------------------- | ------------- | ----------------------- |
| Consume     | Fetched N artifacts        | ids...        | Source: file/inline/web |
| Internalize | Clustered into K themes    | cluster names | Quality distribution    |
| Edit        | Fire X / Water Y / Paper Z | ids...        | Rationale per op        |
| Give Back   | Zine proposal              | top 5 ids     | POST endpoint + tags    |
