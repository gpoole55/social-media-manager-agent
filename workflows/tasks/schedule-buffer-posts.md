# Schedule Buffer Posts — Task Workflow

## Overview

Publishing a social post is a **two-phase flow**, both phases triggered by ClickUp:

| Phase | Trigger | What Happens |
|-------|---------|--------------|
| **Phase 1 — Draft** | ClickUp tag `generate-social-post` added to a task | Agent generates post copy, creates Buffer drafts, creates ClickUp tasks (one per platform), assigns Garrett, sets status → `review` |
| **Phase 2 — Publish** | ClickUp task status changes to `approved` | Agent promotes the Buffer draft to the queue |

Buffer's posting schedule is already configured (Mon/Wed/Fri 10:00 AM MST for FB and LI). **Never manually schedule a time** — just add to queue and Buffer handles it.

---

## Phase 1 — Draft + ClickUp Task Creation

### Inputs (from the triggering ClickUp task)
- Topic / content direction (from task name or description)
- Platform(s) to post to
- Any asset references (video, photo, or "generate infographic")

### Steps

**1. Generate post copy**
Follow `generate-engagement-post.md` to produce platform-specific copy for each target channel.

**2. Create Buffer draft for each platform**
```bash
.venv/bin/python tools/buffer_publish.py draft <channel> "<caption>" [--image URL] [--video URL]
```
Save the Buffer post ID returned — you'll need it in the ClickUp task description.

**3. Create one ClickUp task per platform post**

List: Content Queue (`901414572627`)

Task naming convention:
```
[Platform] — [Topic] — [Date]
```
Examples:
- `Facebook — Key Cards vs. Physical Keys — Apr 22`
- `LinkedIn — Key Cards vs. Physical Keys — Apr 22`

Required fields on each task:
- **Status:** `review`
- **Assignee:** Garrett Poole (user ID `90278850`)
- **Tag:** `social-media`
- **Agent custom field** (`20572024-c406-4299-91b3-2a4934837a7a`): Social Media Manager (`5d30dbd1-f0a8-42e4-b49b-2b8057b691c5`)
- **Description (markdown):** Use the template below

Task description template:
```markdown
## Draft Ready for Review

**Platform:** [Facebook / LinkedIn / Instagram]
**Buffer Draft ID:** [post_id from buffer_publish.py output]
**Buffer Drafts Page:** https://publish.buffer.com/profile/[channel_id]/queue/drafts

**Caption:**
---
[Full caption text]
---

**Asset Used:** [filename or "infographic" + description]
**Rationale:** [1-2 sentences on why this asset and angle]

---
_To approve: change task status to **Approved**_
_To request changes: leave a comment_
```

Buffer channel IDs for the drafts page link:
- Facebook: `69dd1a1d031bfa423cfca01e`
- LinkedIn: `69dd1ba5031bfa423cfca620`
- Instagram: `69dd1a05031bfa423cfc9fbd`

**4. Verify tasks were created**
Confirm each task appears in the Content Queue with status `review` and Garrett assigned.

---

## Phase 2 — Approved → Queue

Triggered when a Content Queue task status changes to `approved`.

### Inputs (from the ClickUp task)
- Buffer Draft ID (from the task description)
- Platform/channel (from the task name)

### Steps

**1. Parse the Buffer Draft ID from the task description**

**2. Promote the draft to the queue**
```bash
.venv/bin/python tools/buffer_publish.py promote <draft_post_id>
```
This queues the post for the next available slot in Buffer's schedule. No manual time needed.

**3. Verify the post is queued**
```bash
.venv/bin/python tools/buffer_publish.py posts --channel <channel>
```
Confirm it shows `status: scheduled` or `status: buffer` in the queue.

**4. Update the ClickUp task**
- Add a comment: `✅ Post added to Buffer queue. Will publish at next scheduled slot (Mon/Wed/Fri 10:00 AM MST).`
- Set status → `complete`

---

## Tool Reference

```bash
.venv/bin/python tools/buffer_publish.py channels                         # List all channels + status
.venv/bin/python tools/buffer_publish.py draft <channel> "<text>"         # Create draft
.venv/bin/python tools/buffer_publish.py draft <channel> "<text>" --image URL
.venv/bin/python tools/buffer_publish.py draft <channel> "<text>" --video URL
.venv/bin/python tools/buffer_publish.py promote <post_id>                # Draft → queue
.venv/bin/python tools/buffer_publish.py posts --channel <channel>        # Inspect queue
.venv/bin/python tools/buffer_publish.py delete <post_id>                 # Remove a draft
```

**Channel shortcuts:** `fb`, `li`, `ig`, `yt`, `gbp-phoenix`, `gbp-chandler`, `gbp-arcadia`

`BUFFER_TOKEN` is read from `.env` automatically.

---

## Edge Cases

- **Auth error (401):** `BUFFER_TOKEN` in `.env` has expired. Ask Garrett to generate a new one at https://publish.buffer.com/settings/api. Save the draft caption in the ClickUp task so no work is lost.
- **Channel shows DISCONNECTED:** Platform needs reconnecting in Buffer. Flag for Garrett, hold the draft.
- **Instagram posts require an image asset.** Create the draft as text-only, note in the ClickUp task that an image must be attached in Buffer's UI before approval.
- **Garrett leaves a revision comment instead of approving:** Read the comment, delete the old Buffer draft (`delete <post_id>`), make changes, create a new draft, update the task description with the new Draft ID. Keep status at `review`.
- **promote returns "not a draft" error:** The draft may have already been promoted or deleted. Run `posts --channel <channel> --status draft` to check. If gone, re-create from the caption in the task description.
