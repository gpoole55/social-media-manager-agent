# Schedule Buffer Posts — Task Workflow

All Buffer operations use the **Buffer MCP connector**. No Python scripts, no API keys — the routine's MCP connection handles auth.

Buffer organization ID: `69dd19b9c941c3b168a916c6`
Channel IDs: Facebook `69dd1a1d031bfa423cfca01e`, LinkedIn `69dd1ba5031bfa423cfca620`, Instagram `69dd1a05031bfa423cfc9fbd`

## Overview

Publishing a social post is a **two-phase flow**, both phases triggered by ClickUp task status changes:

| Phase | Trigger | What Happens |
|-------|---------|--------------|
| **Phase 1 — Draft** | Task is `open` / `to do` with tag `social-media` | Agent generates post copy, creates Buffer drafts (via Buffer MCP), creates ClickUp subtasks (one per platform), assigns Garrett, sets each to `review` |
| **Phase 2 — Publish** | Subtask status → `approved` | Agent promotes the Buffer draft to the queue (via Buffer MCP) |

Buffer's posting schedule is already configured (Mon/Wed/Fri 10:00 AM MST for FB and LI). **Never manually schedule a time** — add to queue and Buffer handles it.

---

## Phase 1 — Draft + ClickUp Subtask Creation

### Inputs (from the triggering ClickUp task)
- Topic / content direction (from task name or description)
- Platform(s) to post to
- Any asset references (video ID, photo ID, or "generate infographic")

### Steps

**1. Generate post copy**
Follow `generate-engagement-post.md` to produce platform-specific copy for each target channel.

**2. Create a Buffer draft for each platform — via Buffer MCP**

Use the Buffer MCP's `create_post` tool with:
- `organizationId`: `69dd19b9c941c3b168a916c6`
- `channelId`: the channel ID from the table above
- `text`: the full caption
- `video` OR `image`: the asset URL (Drive download URL for videos, Canva PNG export URL or stock URL for images)
- `saveToDraft`: `true`
- `mode`: `addToQueue`

The response includes an `id` — this is the **Buffer Draft ID**. Save it for the subtask description.

If the Buffer MCP's direct tools don't cover a specific operation, fall back to `introspect_schema` + `execute_mutation`.

**3. Create one ClickUp subtask per platform — via ClickUp MCP**

Use `clickup_create_task` with:
- `list_id`: `901414572627` (Content Queue)
- `parent`: the triggering task ID (makes this a subtask)
- `name`: `[Platform] — [Video | Graphic] — [Topic]` (e.g. `Facebook — Video — Key Cards vs. Physical Keys`)
- `status`: `review`
- `assignees`: `["90278850"]` (Garrett)
- `tags`: `["social-media"]`
- `custom_fields`: `[{"id": "20572024-c406-4299-91b3-2a4934837a7a", "value": "5d30dbd1-f0a8-42e4-b49b-2b8057b691c5"}]` (routes back to Social Media Manager on approval)
- `markdown_description`: use the template below

```markdown
## Draft Ready for Review

**Platform:** [Facebook | LinkedIn | Instagram]
**Post type:** [Video | Canva Graphic]
**Buffer Draft ID:** <id>
**Buffer Drafts Page:** https://publish.buffer.com/profile/<channel_id>/queue/drafts

**Caption:**
---
<full caption>
---

**Asset:** <filename> (<drive_file_id or catalog id>)
**Canva Design ID:** <design_id>  _(graphics only)_
**Rationale:** <1-2 sentences on why this asset + angle>

---
_To approve: change task status to **Approved**._
_To request changes: leave a comment._
```

**4. Post a summary comment on the parent task**

Use `clickup_create_task_comment` on the parent task with:

```
Generated 2 social media drafts:

✅ [Subtask 1 name] — task <subtask_id>, Buffer draft <draft_id>
✅ [Subtask 2 name] — task <subtask_id>, Buffer draft <draft_id>
```

If one path failed, mark it ❌ with the failure reason.

**5. Set the parent task to `Complete`** using `clickup_update_task`.

---

## Phase 2 — Approved → Queue

Triggered when a subtask status changes to `approved`.

### Steps

**1. Parse the Buffer Draft ID from the subtask description.**

**2. Promote the draft — via Buffer MCP**

Use the Buffer MCP to flip the draft from `saveToDraft: true` to `saveToDraft: false` with `mode: addToQueue`. The exact tool depends on what the MCP exposes:
- If there's a `promoteDraft` or `updatePost` tool, use it directly.
- Otherwise, `introspect_schema` and call the appropriate mutation via `execute_mutation`.

No manual scheduling time — Buffer's schedule queues it automatically.

**3. Verify it's queued**

Query the channel's queued posts and confirm the post now appears with `status: scheduled` (or equivalent). Optional but good hygiene.

**4. Update the ClickUp subtask**

- Comment: `✅ Post added to Buffer queue. Will publish at next scheduled slot (Mon/Wed/Fri 10:00 AM MST).`
- Set status → `complete` via `clickup_update_task`.

---

## Edge Cases

- **Buffer MCP returns a permission error.** The routine's Buffer MCP connection may have become stale. Surface the full error in a ClickUp comment on the parent task and stop. Do not attempt a fallback — Garrett will re-authorize the connector.
- **Channel shows DISCONNECTED.** Platform needs reconnecting in Buffer's UI. Flag in a ClickUp comment, hold the draft.
- **Instagram posts require an image asset.** If the plan came in without one, stop before drafting and flag — don't create a text-only IG draft.
- **Garrett leaves a revision comment instead of approving:** the orchestrator handles this flow (delete old draft, regenerate, update subtask). See the main `social-media-manager.md` orchestration notes.
- **Promote fails because the draft is already queued/sent:** log the state and proceed (don't re-promote). Comment on the subtask with what you found.
