# Generate Engagement Post — Task Workflow

## Objective

Generate a single social media engagement post for a specific topic and platform.

All data access is through MCP connectors (Supabase, Canva, ClickUp, Buffer) — no Python scripts, no API keys, no `.env` file.

## Required Inputs

| Input | Source | Example |
|-------|--------|---------|
| **Topic** | From the triggering ClickUp task description | `Key cards vs. physical keys` |
| **Platform** | `facebook` or `linkedin` (or both, per the brief) | `linkedin` |

## Steps

### 1. Read Brand Guidelines + Style References

- `/tmp/shared/anderson-lock-and-safe-ai-guidelines.md` — tone, values, restricted content
- `/tmp/shared/brand/style_guide.md` — design tokens, named programs to surface
- `/tmp/shared/brand/style_references.md` — common-pattern checklist (thumbnail-recognizable, named program surfaced, benefit-led first line, commercial framing, specific, caption + graphic reinforce each other)

### 2. Search the Content Catalog — via Supabase MCP

Project: `fujaqozmdebcreqpzjcv` (Content Manager). The `document` column on each row is a plain-text Gemini summary — use SQL `ILIKE` on it, not vector search.

**Video candidates:**
```sql
SELECT id, filename, folder_month, duration_sec, orientation, web_content_link, document
FROM assets_video
WHERE folder_month >= '2025-11'
  AND (document ILIKE '%<topic keyword 1>%' OR document ILIKE '%<topic keyword 2>%')
ORDER BY folder_month DESC
LIMIT 10;
```

**Photo candidates (for Canva graphics):**
```sql
SELECT id, filename, folder_month, drive_file_id, tags, document
FROM assets_photo
WHERE source = 'drive'
  AND (document ILIKE '%<topic>%' OR 'technician' = ANY(tags) OR 'truck' = ANY(tags))
ORDER BY folder_month DESC
LIMIT 10;
```

**Collision check** (don't repeat a post from the last 60 days on the same platform):
```sql
SELECT platform, published_at, left(document, 150) as preview
FROM copy_docs
WHERE source_type = 'buffer_post'
  AND platform = '<facebook|linkedin>'
  AND published_at > now() - interval '60 days'
  AND document ILIKE '%<topic>%'
LIMIT 5;
```

If a collision exists, shift the angle or flag and stop. Past captions are structural references only — never copy text.

### 3. Write the Caption

**Facebook posts should:**
- Be conversational, first-person, relatable
- Use casual language ("we've all been there", "raise your hand if...")
- Include a clear question to drive comments
- 1–3 short paragraphs max
- 0 hashtags, ≤1 emoji
- End with a question or call to engage

**LinkedIn posts should:**
- Lead with expertise and industry knowledge
- Reference specific commercial contexts (property management, facilities, multi-unit)
- Mention Anderson Lock and Safe's 60+ year track record where natural
- Be professional but not stiff
- 2–4 paragraphs
- 5–7 relevant branded hashtags at the end
- End with a question to drive professional discussion

Reference **a specific detail** from the catalog asset (technician name, equipment, location from the Gemini `document` summary). Generic captions fail the quality check.

### 4. (If graphic) Generate via Canva MCP

If the plan is a Canva graphic (not a video):

**4a. Upload the photo to Canva** (or reuse cached asset_id):
- Check Supabase first:
  ```sql
  SELECT canva_asset_id FROM assets_photo WHERE id = 'drive:photo:<file_id>';
  ```
- If null, call Canva MCP `upload-asset-from-url` with `url = https://lh3.googleusercontent.com/d/<drive_file_id>=s2000` (this direct-serve URL works; `drive.google.com/uc` does NOT).
- Cache the returned `asset_id`:
  ```sql
  UPDATE assets_photo
  SET canva_asset_id = '<asset_id>', canva_uploaded_at = now()
  WHERE id = 'drive:photo:<file_id>';
  ```

**4b. Generate the design** with Canva MCP `generate-design`:
- `design_type`: `instagram_post` (or `facebook_post`)
- `brand_kit_id`: `kAGLyB_BxbM`
- `asset_ids`: `["<asset_id>"]`
- `query`: detailed creative brief including `"Anderson Lock and Safe, a commercial locksmith in Phoenix, Arizona, founded 1966 (60+ years)."`, headline, body, `"USE THE ATTACHED REAL ANDERSON PHOTO — NO STOCK OR AI IMAGERY."`, logo placement, colors `#0045DB` / `#141A2E`, tagline `"Securing Arizona Since 1966."`

**4c.** `create-design-from-candidate` → `export-design(format={type:'png',export_quality:'regular'})` → capture the PNG download URL.

Run the common-pattern checklist from `/tmp/shared/brand/style_references.md` before using the graphic.

### 5. Format the Output

```
PLATFORM: [Facebook / LinkedIn]
TOPIC: [Topic]

---

[Post copy here]

---

HASHTAGS: [LinkedIn only — 5–7 hashtags]
ASSET: [filename + catalog ID OR Canva design ID + PNG URL]
```

### 6. Quality Check

Before passing to the Buffer step, verify:
- [ ] Does NOT lead with a hard sell or CTA
- [ ] Asks a genuine question that invites engagement
- [ ] Demonstrates expertise through specificity (references a real asset detail)
- [ ] Matches the platform tone (casual for FB, professional for LI)
- [ ] PHOENIX / ARIZONA ONLY — never Chicago or any other city
- [ ] Does NOT focus on residential locksmithing
- [ ] Does NOT compete on price
- [ ] Length is appropriate (FB: shorter/punchier, LI: can be longer)

## Next Step

Hand the output to [schedule-buffer-posts.md](schedule-buffer-posts.md) to create the Buffer draft (via Buffer MCP) and the ClickUp review subtask.

## Edge Cases

- **If the catalog has no matching asset,** don't fall back to stock — surface in a ClickUp comment that content is needed and stop. The orchestrator decides whether to proceed without a video or shift angle.
- **If generating for both platforms at once,** produce two separate posts with distinct angles — never just copy/paste and adjust tone.
- **If Supabase returns zero rows for the collision query,** proceed confidently. Empty = no collision.
