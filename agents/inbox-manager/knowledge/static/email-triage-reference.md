# Email Triage Reference

Quick-reference card for the Inbox Manager agent.

---

## Categorization Rules

Every incoming email must be classified into exactly one category:

| Category | Criteria | Action |
|----------|----------|--------|
| **URGENT** | From a priority domain, contains deadline language ("ASAP", "by EOD", "emergency"), or is a reply to a flagged thread | Flag to Chief of Staff immediately with full context |
| **NEEDS-ATTENTION** | Requires a thoughtful reply, contains a question with nuance, or involves scheduling/commitments | Flag to Chief of Staff with a suggested response approach |
| **ROUTINE** | Standard replies, confirmations, meeting accepts, simple questions with clear answers | Draft a response for approval — never send without explicit sign-off |
| **INFORMATIONAL** | Newsletters, notifications, receipts, automated reports, FYI-only forwards | Summarize in one line and file; no response needed |
| **SPAM/IGNORE** | Marketing blasts, unsolicited sales, phishing attempts, irrelevant mailing lists | Log as ignored; do not surface unless from a priority domain |

**Decision priority:** If an email matches multiple categories, use the highest-action category (URGENT > NEEDS-ATTENTION > ROUTINE > INFORMATIONAL > SPAM).

---

## Priority Domain Matching

Priority domains are configured via the `PRIORITY_DOMAINS` environment variable (comma-separated).

- Match against the sender's email domain (everything after the `@`)
- Subdomains count: if `bigclient.co` is listed, `invoices.bigclient.co` also matches
- Any email from a priority domain is automatically elevated to at least NEEDS-ATTENTION
- If an email from a priority domain also meets URGENT criteria, classify as URGENT

---

## Draft Response Templates

### Routine Acknowledgment

```
Hi {first_name},

Thanks for sending this over. {one-sentence confirmation of what was received}.

{one-sentence next step if applicable, otherwise remove this line}

Best,
{{USER_NAME}}
```

### Meeting Confirmation

```
Hi {first_name},

Confirmed for {day, date} at {time} {timezone}. {Location or link if provided.}

See you then.

Best,
{{USER_NAME}}
```

### Simple Question Reply

```
Hi {first_name},

{Direct answer in 1-2 sentences.}

{If the answer requires a caveat or follow-up, add it here.}

Best,
{{USER_NAME}}
```

### Decline / Not Now

```
Hi {first_name},

Thanks for thinking of me. {Brief, honest reason — keep it to one sentence.}

{Optional: suggest an alternative or future timing.}

Best,
{{USER_NAME}}
```

---

## Running Inbox Summary Schema

Maintain a running summary in `knowledge/dynamic/inbox-summary.md`:

```markdown
# Inbox Summary

## Last Scan
- timestamp: {ISO 8601}
- new_emails: {count}

## Pending Review
{list of emails flagged to Chief of Staff, with subject and category}

## Drafts Awaiting Approval
{list of drafted responses waiting for user sign-off}

## Today's Stats
- urgent: {count}
- needs_attention: {count}
- routine: {count}
- informational: {count}
- spam: {count}
```

---

## Escalation Rules

1. Never send any email without explicit user approval
2. If unsure about categorization, escalate to NEEDS-ATTENTION rather than guessing
3. If an email thread has been flagged URGENT and no response has been sent within 2 scan cycles (10 minutes), re-flag to Chief of Staff with an urgency reminder
4. If the same sender appears 3+ times in one day at ROUTINE or above, note the pattern to Chief of Staff
