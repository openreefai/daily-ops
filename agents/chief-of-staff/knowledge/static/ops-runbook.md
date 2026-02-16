# Ops Runbook

Quick-reference card for the Chief of Staff agent.

---

## Formation Topology

```
                    ┌──────────────────┐
                    │  chief-of-staff  │   (coordinator - cron: */15)
                    └──┬──┬──┬──┬─────┘
                       │  │  │  │
          ┌────────────┘  │  │  └────────────┐
          ▼               ▼  ▼               ▼
  ┌───────────────┐ ┌──────────────┐ ┌───────────────┐
  │ inbox-manager │ │   research   │ │content-writer │
  │  (triage)     │ │   analyst    │ │   (writer)    │
  └───────┬───────┘ └──────┬───────┘ └───────┬───────┘
          │                │                 │
          └────────────────┴─────────────────┘
                           │ results back
                           ▼
                    ┌──────────────────┐
                    │  chief-of-staff  │
                    └────────┬─────────┘
                             │ compiled inputs
                             ▼
                    ┌──────────────────┐
                    │ daily-briefing   │   (compiler - cron: 0 7 * * *)
                    └──────────────────┘
```

**Edges (agentToAgent):**
- `chief-of-staff → inbox-manager` - email review tasks, priority overrides
- `chief-of-staff → research-analyst` - research requests with scope and deadline
- `chief-of-staff → content-writer` - content tasks with topic, format, and voice
- `chief-of-staff → daily-briefing` - compiled daily activity for synthesis
- `inbox-manager → chief-of-staff` - urgent flags, draft responses for review
- `research-analyst → chief-of-staff` - completed research briefs
- `content-writer → chief-of-staff` - content drafts for review
- `daily-briefing → chief-of-staff` - compiled morning briefing

No direct communication between specialists. All work flows through Chief of Staff.

---

## Task Board Schema

Maintain a running task board in `knowledge/dynamic/task-board.md`:

```markdown
# Task Board

## Active Tasks
| ID | Agent | Type | Summary | Status | Created | Updated |
|----|-------|------|---------|--------|---------|---------|
| T-001 | research-analyst | research | {topic} | in-progress | {ISO} | {ISO} |

## Completed Today
| ID | Agent | Type | Summary | Completed |
|----|-------|------|---------|-----------|

## Blocked / Needs Escalation
| ID | Agent | Reason | Flagged |
|----|-------|--------|---------|
```

---

## Priority Hierarchy

| Priority | Criteria | Response Time |
|----------|----------|---------------|
| **P0 - Critical** | Urgent email from priority domain, time-sensitive commitment, system outage | Immediate: surface to user within current cycle |
| **P1 - High** | Needs-attention emails, pending approvals blocking work, overdue tasks | Within 2 heartbeat cycles (30 minutes) |
| **P2 - Normal** | Research requests, content tasks, routine delegations | Within the day |
| **P3 - Low** | Informational items, background research, nice-to-have content | When capacity allows |

---

## Delegation Rules

1. **One task per message.** Each delegation to a specialist should contain exactly one task with clear scope
2. **Include context.** Always provide: what is needed, why it matters, any constraints, and the deadline
3. **Match agent to task.** Use the right specialist:
   - Email questions → Inbox Manager
   - Deep research → Research Analyst
   - Written output → Content Writer
   - Morning synthesis → Daily Briefing
4. **Track everything.** Every delegated task gets a task board entry
5. **Follow up.** If a task has been in-progress for more than 2 heartbeat cycles without an update, nudge the agent
6. **Never bypass specialists.** Even if you could draft a response yourself, delegate to maintain clean separation

---

## Heartbeat Checklist (*/15 cron)

Run through this checklist on every heartbeat cycle:

1. **Check Inbox Manager flags.** Any urgent items? Review and decide: handle, escalate to user, or delegate
2. **Review task board.** Any tasks stuck or overdue? Nudge the responsible agent
3. **Check completed work.** Any drafts or briefs ready for user review? Surface them
4. **Synthesize progress.** If multiple items have completed since last heartbeat, write a brief status update
5. **Update task board.** Mark completed items, update statuses, add new tasks
6. **Context accumulation.** Note any patterns in user preferences, recurring topics, or priority shifts

---

## Delegation Message Template

When delegating to a specialist, include all of these fields:

```markdown
## Task

- **Task ID:** {T-NNN}
- **Type:** {research | content | email-review | briefing}
- **Priority:** {P0 | P1 | P2 | P3}
- **Deadline:** {ISO timestamp or "end of day" or "next heartbeat"}

### Request

{Clear description of what is needed}

### Context

{Why this matters, any relevant background}

### Constraints

{Word count, tone, format requirements, or "none"}
```
