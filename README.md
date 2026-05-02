# Curation As Craft Skill
a creative partner agent 

**tldr: started at an existential crisis...ended up with a skill.md** 

instead of waiting for ai to replace me
I decided to teach it how 

started with an essay titled 
curation as craft (available in the references folder)
about my creative practice 
trying to get to the core of
why I make things
 
this is the resulting ai agent skill
experiment with it as a creative partner

# User Guide

## Activate the skill

Say any of the following:

- "Curate a zine from my canvas"
- "Remix these artifacts as a blog / post / gallery"
- "Curation suggestions for [theme]"
- "What should I cut / keep / publish?"
- "Cluster these artifacts"
- "Run the atelier pipeline on X"

---

## Provide your artifacts

Upload a JSON or CSV file, paste data inline, or describe your canvas.

**Required fields:** `id`, `content`, `quality_score`
**Optional fields:** `tags`, `author`, `type`, `op_time`

No file? Paste a JSON array or describe your canvas — the skill will ask for what it needs.

**Minimum example:**
```json
[
  { "id": "a1", "content": "generative art with diffusion models", "quality_score": 0.9 },
  { "id": "a2", "content": "typography for editorial zine layouts", "quality_score": 0.85 },
  { "id": "a3", "content": "generative art prompt engineering", "quality_score": 0.42 }
]
```

---

## The 4 phases

**01 Consume** — Fetches artifacts from your file, inline data, or canvas description.

**02 Internalize** — Clusters artifacts by theme, scores into tiers: `high` / `mid` / `low`. Runs automatically, no setup needed.

**03 Edit** — Applies one of 4 ops per artifact:

| Op | Trigger | Action |
|----|---------|--------|
| Water | Low-quality, not a duplicate | Flagged for enrichment |
| Fire | Duplicate cluster + score < 0.5 | Marked for deletion |
| Paper | Gap in theme coverage | AI generates a new artifact |
| Scissors | High-tier, strong cluster signal | Selected for compilation |

**04 Give Back** — Produces one of:

- **Zine/blog proposal** — title, ordered artifact ids, tags, POST endpoint payload
- **Gallery impact estimate** — reach prediction based on cluster diversity
- **Diff snapshot** — fired / watered / added ids with rationale per op
