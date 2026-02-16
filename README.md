# Daily Ops

> Your operational layer, orchestrated.

A hub-and-spoke formation for daily business operations. You talk only to the Chief of Staff — four specialist agents handle email triage, deep research, content writing, and daily synthesis behind the scenes. Install once, let it run indefinitely.

## What It Does

The formation runs four continuous loops:

1. **Email loop:** Inbox Manager scans email every 5 minutes → categorizes and drafts routine responses → flags urgent items to Chief of Staff → you review and approve
2. **Research loop:** You request research through Chief of Staff → Research Analyst produces a structured brief → Chief of Staff delivers the results
3. **Content loop:** Content Writer checks the content calendar daily → drafts anything due → flags overdue items to Chief of Staff → you review and publish
4. **Briefing loop:** Daily Briefing compiles the previous 24 hours of activity from all agents into a single morning summary → delivered via your configured channel

All output is reviewed by a human before sending. No agent publishes or sends anything autonomously.

## Requirements

- [OpenClaw](https://github.com/openclaw/openclaw) >= 0.5.0 installed and running
- [OpenReef CLI](https://github.com/openreef/openreef) installed
- Anthropic API key (for Chief of Staff, Research Analyst, Content Writer)
- Google AI API key (for Inbox Manager, Daily Briefing)
- Email access (IMAP or API) for the Inbox Manager

## Variables

| Variable | Type | Default | Required | Description |
|----------|------|---------|----------|-------------|
| `USER_NAME` | string | — | yes | Your name, used in drafts and briefings |
| `BUSINESS_NAME` | string | — | yes | Your company or business name |
| `BRIEFING_CHANNEL` | string | `slack` | no | Channel for daily briefing delivery (slack, telegram, or whatsapp) |
| `WRITING_VOICE` | string | `Direct, conversational, no jargon. Writes like a smart person talking to another smart person.` | no | Description of your writing voice/tone for the Content Writer |
| `PRIORITY_DOMAINS` | string | `""` | no | Comma-separated email domains that should always be flagged as important (e.g. investor.com,bigclient.co) |

## Quick Start

```bash
# 1. Copy and fill in your environment variables
cp .env.example .env
# Edit .env — set USER_NAME and BUSINESS_NAME at minimum

# 2. Lock skill versions
reef lock .

# 3. Deploy the formation
reef install .
```

Once installed, talk to the Chief of Staff via Slack or Telegram. Everything routes through that single point of contact.

## Agents

### Chief of Staff

**Model:** `anthropic/claude-opus-4-6` | **Role:** Coordinator | **Cron:** `*/15 * * * *`

Single point of contact for the entire formation. Orchestrates all delegation, tracks tasks on an internal task board, synthesizes progress across all agents, and accumulates long-term context about your priorities and preferences. Runs a heartbeat check every 15 minutes to review the task board, nudge stuck items, and surface completed work.

### Inbox Manager

**Model:** `google/gemini-3-flash-preview` | **Role:** Triage | **Cron:** `*/5 * * * *`

Monitors email on a 5-minute cycle. Categorizes every incoming message as urgent, needs-attention, routine, informational, or spam. Drafts responses for routine items using your configured voice. Flags anything urgent to the Chief of Staff immediately. Never sends without explicit approval.

### Research Analyst

**Model:** `anthropic/claude-opus-4-6` | **Role:** Researcher

On-demand deep research, triggered by the Chief of Staff. Produces structured briefs with an executive summary, key findings, sources, and recommended actions. No cron schedule — activated only when a research task is delegated.

### Content Writer

**Model:** `anthropic/claude-opus-4-6` | **Role:** Writer | **Cron:** `0 8 * * *`

Handles all written output: blog posts, social media, email campaigns, investor updates, and memos. Checks the content calendar daily at 8 AM ET. Drafts anything due, flags overdue items to the Chief of Staff, and reports all-clear if nothing is scheduled.

### Daily Briefing

**Model:** `google/gemini-3-flash-preview` | **Role:** Compiler | **Cron:** `0 7 * * *`

Compiles the previous 24 hours of activity from all agents into a single structured morning briefing. Covers what was handled, what is outstanding, what needs attention today, and includes a 3-line executive summary at the top. Delivers via the configured briefing channel at 7 AM ET.

## Topology

```
inbox-manager ⇄ chief-of-staff → research-analyst
                                → content-writer
                                → daily-briefing
```

**Explicit edge list:**

| From | To |
|------|----|
| chief-of-staff | inbox-manager, research-analyst, content-writer, daily-briefing |
| inbox-manager | chief-of-staff |
| research-analyst | chief-of-staff |
| content-writer | chief-of-staff |
| daily-briefing | chief-of-staff |

Chief of Staff is the hub. All specialist agents communicate exclusively through Chief of Staff — no direct communication between specialists. The user interacts only with Chief of Staff via Slack or Telegram bindings.

## Cron Schedule

| Agent | Schedule | Description |
|-------|----------|-------------|
| Inbox Manager | `*/5 * * * *` | Every 5 minutes |
| Chief of Staff | `*/15 * * * *` | Every 15 minutes (heartbeat check) |
| Content Writer | `0 8 * * *` | Daily at 8:00 AM |
| Daily Briefing | `0 7 * * *` | Daily at 7:00 AM |

All schedules use `America/New_York` timezone. Adjust the `timezone` field in `reef.json` if you operate in a different zone.

## How It Works

### Email Triage Loop

1. Every 5 minutes, the Inbox Manager polls for new emails
2. Each message is categorized: urgent, needs-attention, routine, informational, or spam
3. Routine emails get a draft response (using your voice and templates)
4. Urgent and needs-attention items are flagged to the Chief of Staff
5. On the next heartbeat, Chief of Staff reviews flags and decides: handle directly, escalate to the user, or delegate to a specialist
6. Nothing is ever sent without your explicit approval

### Research Loop

1. You ask the Chief of Staff a research question (via Slack or Telegram)
2. Chief of Staff delegates to the Research Analyst with scope and context
3. Research Analyst produces a structured brief: executive summary, key findings, sources, recommended actions
4. Brief is returned to Chief of Staff, who presents it to you

### Content Loop

1. At 8 AM ET, the Content Writer checks the content calendar
2. Items due today are drafted in your configured writing voice
3. Overdue items are flagged to the Chief of Staff with a status update
4. Drafts are surfaced for your review on the next heartbeat cycle

### Briefing Loop

1. At 7 AM ET, the Daily Briefing agent pulls the previous 24 hours of activity
2. It compiles inputs from all four specialists: emails handled, research completed, content produced, tasks managed
3. The result is a single structured briefing with a 3-line executive summary
4. Delivered via your configured briefing channel (Slack, Telegram, or WhatsApp)

## Channel Bindings

| Channel | Agent |
|---------|-------|
| Slack | chief-of-staff |
| Telegram | chief-of-staff |

Both channels route to the Chief of Staff. This is the only agent the user interacts with directly. All delegation and coordination happens behind the scenes.

## Teardown

```bash
reef uninstall ops/daily-ops
```

**WARNING:** `reef uninstall` destroys agent workspaces, including all `knowledge/dynamic/` contents — task boards, inbox summaries, research briefs, content drafts, and briefing history are permanently deleted.

Runtime data lives in OpenClaw workspaces, **not** in the source tree:

```
$OPENCLAW_STATE_DIR/workspace-ops-chief-of-staff/knowledge/dynamic/
$OPENCLAW_STATE_DIR/workspace-ops-inbox-manager/knowledge/dynamic/
$OPENCLAW_STATE_DIR/workspace-ops-research-analyst/knowledge/dynamic/
$OPENCLAW_STATE_DIR/workspace-ops-content-writer/knowledge/dynamic/
$OPENCLAW_STATE_DIR/workspace-ops-daily-briefing/knowledge/dynamic/
```

`$OPENCLAW_STATE_DIR` defaults to `~/.openclaw/` if not set.

**Before uninstalling,** copy any data you want to keep out of those workspace directories. The source `formations/daily-ops/` directory is unaffected by uninstall.
