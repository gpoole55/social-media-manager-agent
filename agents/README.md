# Agents — AI Marketing Department

These are the agent prompts for Anderson Lock and Safe's AI marketing team. Each file defines a specialist role that can be run as a Claude Routine, an on-demand Claude Code session, or triggered from ClickUp.

## The Team

| Agent | File | Primary Tools | ClickUp Assignment |
|-------|------|--------------|-------------------|
| **PPC Specialist** | `ppc-specialist.md` | Google Ads, Bing Ads, Meta Ads, Windsor.ai | PPC Specialist |
| **Social Media Manager** | `social-media-manager.md` | Buffer (`buffer_publish.py`), ClickUp | Social Media Manager |
| **Email Marketing Specialist** | `email-marketing-specialist.md` | Klaviyo, ServiceTitan | Email Marketing Specialist |
| **Content Strategist** | `content-strategist.md` | All tools (orchestrates content kits) | Content Strategist |
| **Lead Gen Specialist** | `lead-gen-specialist.md` | Apollo, ServiceTitan | Lead Gen Specialist |

## How It Works

### The Loop

```
1. Garrett assigns a task in ClickUp to an agent role
2. Agent picks up the task (via Routine, webhook, or manual trigger)
3. Agent reads the task description + its WAT workflow
4. Agent executes the work using MCP tools and Python scripts
5. Agent posts results as a ClickUp task comment
6. Agent sets task status to "Review" (needs Garrett) or "Complete"
7. Garrett reviews, approves, or redirects
```

### Running an Agent

**As a Claude Routine (scheduled/autonomous):**
The Routine's system prompt is the agent file contents. It clones this repo, reads its workflows, checks ClickUp for assigned tasks, and executes.

**As an on-demand Claude Code session:**
```
"You are the PPC Specialist. Read agents/ppc-specialist.md for your identity and instructions. Check ClickUp for your assigned tasks and get to work."
```

**Triggered by ClickUp webhook (future):**
ClickUp automation fires on task assignment → webhook hits Routine API → agent runs for that specific task.

### Agent Boundaries

Every agent has explicit "CAN do" and "MUST get approval" sections. The general rule:

- **Read, analyze, report, draft** → Agents can do freely
- **Publish, send, spend, change, delete** → Needs Garrett's approval

This keeps Garrett in the strategist seat while agents handle execution.

## Relationship to WAT Framework

```
agents/                  → WHO does the work (identity + instructions)
workflows/               → WHAT to do (SOPs and procedures)
workflows/tasks/         → HOW to do it (step-by-step for specific operations)
tools/                   → The deterministic scripts that execute API calls
```

An agent reads its identity, finds its assigned task, looks up the relevant workflow, and uses the tools to execute.

## ClickUp Integration

Tasks are assigned to agents via a **custom dropdown field** called "Agent" on the Marketing Department Agent space.

**Custom Field ID:** `20572024-c406-4299-91b3-2a4934837a7a`

| Agent | Option ID | Color |
|-------|-----------|-------|
| PPC Specialist | `2bb6a2cf-31bf-4cf6-af84-0f72892c57e6` | 🔵 Blue |
| Social Media Manager | `5d30dbd1-f0a8-42e4-b49b-2b8057b691c5` | 🟢 Green |
| Email Marketing Specialist | `6d7f23c3-1578-430c-8076-d08a962b4ab1` | 🟣 Purple |
| Content Strategist | `0a216f71-47ad-4fe4-b7be-e48f26131b05` | 🟠 Orange |
| Lead Gen Specialist | `117d113e-ef08-4ec2-9d14-951d20c12a79` | 🔴 Red |

### Trigger Flow

```
1. Set the "Agent" dropdown on any task in the Marketing space
2. ClickUp Automation detects the field change
3. Automation fires a webhook to the matching Claude Routine
4. Routine clones this repo, loads the agent prompt from agents/
5. Routine reads the task, executes the WAT workflow, writes results back
```

### Setting Up the Automations (In ClickUp UI)

For each agent, create an automation in the Marketing Department Agent space:
- **Trigger:** Custom field "Agent" changes to `[agent name]`
- **Action:** Call webhook → `[Claude Routine API endpoint]`
- **Payload:** Include the task ID so the Routine knows which task to work on
