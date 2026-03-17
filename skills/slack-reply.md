---
name: slack-reply
description: Draft and send Slack replies in the user's tone of voice, with confirmation before sending
triggers:
  - "reply to slack"
  - "respond to"
  - "answer slack"
  - "slack reply"
  - "draft a reply"
  - "check my slack"
  - "what's on slack"
dependencies: []
auto_invoke: false
version: 1.0.0
---

# Slack Reply

Draft Slack replies in the user's tone and send only after explicit confirmation. Supports replying to specific messages, triaging recent mentions, and composing new messages.

## Step 1: Identify the Target Message

The user invokes this skill in one of three ways:

### 1a. Specific message reference

The user points to a message — from a journal `## Slack` entry, a channel name + person, or a quoted snippet. Extract:
- `channel_id` (resolve via `slack_search_channels` if only a channel name is given)
- `message_ts` or `thread_ts` (the message to reply to)

### 1b. Recent mentions triage

The user asks "check my slack" or "what's on slack." Pull recent mentions:

1. Call `slack_search_public_and_private` via MCP (`user-slack`) with:
   - `query`: `"to:me"`
   - `sort`: `"timestamp"`
   - `sort_dir`: `"desc"` (newest first)
   - `limit`: `10`
2. Present a summary list:

```
Recent Slack mentions:
1. #product-analytics — Alex M (2h ago): "Can you review the adoption dashboard before tomorrow's sync?"
2. DM from Jordan P (4h ago): "Quick q about the BI cost analysis"
3. #data-team — Sam K (6h ago): Tagged you in a thread about Snowflake access

Which ones should I draft replies for? (numbers, or "all")
```

3. The user selects which to respond to.

### 1c. New outbound message

The user asks to message someone. Resolve the recipient:
1. Call `slack_search_users` with the person's name
2. Use the `user_id` as the `channel_id` for DMs
3. For channel messages, resolve via `slack_search_channels`

## Step 2: Gather Context

For each message the user wants to reply to:

1. **Thread context** — Call `slack_read_thread` with the `channel_id` and parent `message_ts` to read the full conversation
2. **Sender profile** — Call `slack_read_user_profile` with the sender's `user_id` to understand role, team, timezone
3. **Vault context** — Check for a dossier:
   - Search `team-members/*.md` for direct reports
   - Search `Stakeholders/*.md` for everyone else
   - If a dossier exists, read the `## Log` or `## Interaction Log` for recent history
4. **Recent journal context** — If the topic overlaps with recent journal entries, pull relevant notes for continuity

## Step 3: Draft the Reply

Generate a reply that matches how the user actually communicates on Slack:

### Tone rules

- **Direct, substance-first.** Lead with the answer or decision, not preamble.
- **Concise for operational messages.** Scheduling, acknowledgments, quick answers — keep tight.
- **Thorough for strategic ones.** If the topic requires nuance, give it room.
- **No corporate filler.** No "Hope you're doing well!", "Just circling back", "Per my last message."
- **Casual with direct reports, professional with stakeholders.** Calibrate using the dossier relationship type.
- **Match the language of the conversation.** If the thread is in a non-English language, reply in the same language naturally — not formal/stiff.
- **Emojis sparingly.** A thumbs up or checkmark is fine. No emoji walls.

### Presentation format

Present the draft in chat like this:

```
**Replying to:** @Alex M in #product-analytics (thread from 2h ago)
**Original:** "Can you review the adoption dashboard before tomorrow's sync?"

**Draft reply:**
> Reviewed it — the engagement funnel looks solid but the retention cohorts need a longer lookback window. I'll add comments directly on the dashboard. Should be ready before the sync.

Send this? (yes / edit / skip)
```

For multiple replies, present all drafts sequentially, then ask for batch confirmation.

## Step 4: Confirm and Send

Wait for the user's explicit approval:

| User says | Action |
|-----------|--------|
| "send" / "go" / "yes" / "ship it" | Send via `slack_send_message` with `channel_id` and `thread_ts` |
| "edit: [changes]" | Revise the draft per the user's feedback, re-present |
| "skip" / "nah" | Drop this reply, move to next |
| "draft it" | Save as draft via `slack_send_message_draft` instead of sending |
| "schedule for [time]" | Send via `slack_schedule_message` with the specified `post_at` timestamp |

After sending, confirm delivery:
```
Sent to #product-analytics (thread with Alex M).
```

## Step 5: Edit Loop

If the user requests changes:
1. Apply the edit to the draft
2. Re-present with the same format
3. Wait for confirmation again
4. Never send without explicit OK

## Multi-Reply Flow

When triaging multiple mentions (Step 1b):
1. Gather context for all selected messages in parallel
2. Present all drafts in sequence
3. The user can approve/edit/skip each individually, or say "send all" for batch delivery
4. Execute sends in order, report results

## Edge Cases

### Thread vs. channel reply
Default to in-thread replies (`thread_ts` set). If the user says "reply in channel" or the original message isn't in a thread, omit `thread_ts`.

### Long threads
If a thread has >20 messages, summarize the thread arc in the context presentation rather than showing every message. Still read the full thread for drafting accuracy.

### Unknown person
If the sender has no vault dossier, use their Slack profile (title, team) to calibrate tone. Default to professional-neutral.

### Rate limiting
If sending multiple replies in quick succession, space them 1-2 seconds apart to avoid Slack rate limits.
