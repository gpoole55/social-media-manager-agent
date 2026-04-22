# Generate Engagement Post — Task Workflow

## Objective

Generate a single social media engagement post from the content calendar for a specific date and platform.

## Required Inputs

| Input | Source | Example |
|-------|--------|---------|
| **Date** | From content calendar or user request | `Mon Apr 27` |
| **Platform** | `facebook` or `linkedin` | `linkedin` |
| **Content Calendar** | `engagement-posts-calendar-apr-may-2026.md` | Week 2, Mon Apr 27 row |

## Steps

### 1. Read the Content Calendar
Open `engagement-posts-calendar-apr-may-2026.md` and find the row for the requested date.

Extract:
- **Topic** (e.g., "Key control awareness")
- **Platform-specific direction** (the Facebook column or LinkedIn column)

### 2. Read Brand Guidelines + Style References
- `anderson-lock-and-safe-ai-guidelines.md` — tone, values, restricted content
- `brand/style_guide.md` — design tokens, named programs to surface
- `brand/style_references.md` — common-pattern checklist (thumbnail-recognizable, named program surfaced, benefit-led first line, disciplined stream type, commercial framing, specific, caption + graphic reinforce each other)

### 2.5. Search the Content Catalog
**Before writing anything, search the pre-indexed catalog of Anderson's creative library:**
```bash
.venv/bin/python tools/catalog_content.py search "<topic>" --type video --limit 5
.venv/bin/python tools/catalog_content.py search "<topic>" --type copy --limit 3
```

Use the top-scoring video/photo as your asset (if score >0.55). Use past captions with score >0.6 as structural references — never copy text.

If nothing scores high enough, decide whether the post wants:
- a fresh browse of DropKick Drive (per `browse-content-library.md`)
- a stock photo (`.venv/bin/python tools/stock_photos.py search "<query>"`)
- a branded infographic tile (see Step 3.5 below)

### 3. Generate the Post

**Facebook posts should:**
- Be conversational, first-person, relatable
- Use casual language ("we've all been there", "raise your hand if...")
- Include a clear question to drive comments
- 1-3 short paragraphs max
- No hashtags (or minimal — 1-2 max)
- End with a question or call to engage

**LinkedIn posts should:**
- Lead with expertise and industry knowledge
- Reference specific commercial contexts (property management, facilities, multi-unit)
- Mention Anderson Lock and Safe's 60+ year track record where natural
- Be professional but not stiff
- 2-4 paragraphs
- Include 3-5 relevant hashtags at the end
- End with a question to drive professional discussion

### 3.5. (If infographic) Generate the Graphic

If this post's visual format is a branded tile (tip, stat, Q&A, numbered list, before/after), generate it from brand-aligned tokens:

**Canva (preferred):**
```bash
.venv/bin/python tools/canva_designer.py render --template <id> --vars '{...}' --out .tmp/graphics/post.png
```

**Pillow fallback:**
```bash
.venv/bin/python tools/image_generator.py tip-card --headline "..." --body "..." --out .tmp/graphics/post.png
```

Run the common-pattern checklist from `brand/style_references.md` before saving.

### 4. Format the Output

```
PLATFORM: [Facebook / LinkedIn]
DATE: [Scheduled date]
TOPIC: [Topic from calendar]

---

[Post copy here]

---

HASHTAGS: [LinkedIn only — 3-5 hashtags]
VISUAL DIRECTION: [Brief note on what image would complement this post, if any]
```

### 5. Quality Check

Before finalizing, verify:
- [ ] Does NOT lead with a hard sell or CTA
- [ ] Asks a genuine question that invites engagement
- [ ] Demonstrates expertise through specificity (not generic)
- [ ] Matches the platform tone (casual for FB, professional for LI)
- [ ] Does NOT focus on residential locksmithing
- [ ] Does NOT compete on price
- [ ] Length is appropriate (FB: shorter/punchier, LI: can be longer)
- [ ] For infographic posts: passes the common-pattern checklist in `brand/style_references.md`
- [ ] References a specific catalog hit OR a concrete scenario/service (not generic filler)

## Expected Output

Ready-to-schedule post copy with platform, date, and optional visual direction notes.

## Edge Cases

- **If the calendar entry is vague,** expand it using the topic and the platform tone guidelines. The calendar provides direction, not exact copy.
- **If the date has already passed,** note this and ask whether to generate anyway (for backfill) or skip.
- **If generating for both platforms at once,** produce two separate posts with distinct angles — never just copy/paste and adjust tone.
