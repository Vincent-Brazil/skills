---
name: tone-voice
description: >
  Maintains a persistent Notion page capturing Tom's tone-of-voice patterns, phrasing habits,
  communication preferences, and voice quirks — built from real conversations over time. Use at
  the END of any conversation where a notable writing style choice, phrasing preference, or
  communication pattern emerged that isn't yet documented. Use at the START of any session
  where writing, editing, or communication style is in scope, to orient on the current profile.
  Triggers include: "does that sound like me", "write it in my voice", "capture how I phrase
  things", "update my tone profile", or any task requiring style-matching to Tom's own writing.
compatibility: "Requires Notion MCP connection for reading and updating the tone page."
---

# Tone & Voice Skill

Tom maintains a Notion page as a persistent record of his communication style. This skill
governs how to read it, identify new patterns from conversations, and keep it current.

---

## Core Notion Page

- **Tone & Voice Patterns**: ID `37e75631-9aa7-8192-9539-dc3e689b1e05` `[active]`

Parent: main KB page `35775631-9aa7-815f-ab40-fb0e6d4bf033`

---

## Page Structure

The tone page uses this fixed structure:

- **## What This Is** — brief framing; do not modify
- **## Last Reviewed** — ISO date (`YYYY-MM-DD`); updated whenever the page is written to
- **## Current Patterns** — categorised patterns with subheadings (see categories below)
- **## Open Questions** — unresolved questions about tone; update as answers emerge
- **## Changelog** — append-only table: `| Date | Change | Source |`

Do not add new top-level sections. Patterns go into existing categories or a new subheading
under `## Current Patterns`.

### Pattern categories

| Subheading | What belongs here |
|---|---|
| Directness & Structure | how Tom organises and sequences communication |
| Vocabulary & Register | word choice, formality level, jargon stance |
| Framing & Tone | overall attitude, mood, and perspective |
| Feedback & Critique Style | how Tom gives and expects feedback |
| Writing Conventions | punctuation, capitalisation, list style, mechanics |

Add a new subheading only if a pattern is genuinely orthogonal to all existing categories.

---

## Reading the Page

At the start of any writing or voice-matching session:

1. Fetch the tone page using `Notion:notion-fetch` with ID `37e75631-9aa7-8192-9539-dc3e689b1e05`
2. Note the **Last Reviewed** date — anything in the changelog newer than this date was added
   since the page was last read and should be treated as the most recent signal
3. Load the Current Patterns as the active style guide for the session
4. Do not announce you have read it — just apply it

---

## Identifying New Patterns

Scan the conversation for evidence of patterns not yet on the page:

**What to look for:**
- Phrasing Tom uses repeatedly and consistently (e.g. "the why before the how")
- Corrections Tom makes to Claude's drafts — what he changed signals preference
- Explicit statements of preference ("I never say X", "I prefer Y", "that sounds off")
- Structural choices Tom makes when writing himself (sentence length, list vs. prose)
- Tone mismatches Tom flags — "too formal", "too casual", "not how I'd put it"
- Domain-specific shifts — does Tom write differently in technical contexts vs. strategic ones?

**Signal vs. noise:**
- A single usage is weak evidence. Look for at least two instances, or one explicit statement.
- Context-forced language (e.g. quoting someone else, filling in a template) is not a pattern.
- Deliberate departures ("let's be formal for this one") are not new patterns — they are
  exceptions. Note them in Open Questions if the exception reveals something interesting.

---

## Proposing Updates

At the end of any session where a new pattern was identified:

1. State the pattern in one sentence, cite where in the conversation it appeared
2. Propose the specific addition: which category, the exact wording of the new bullet
3. Wait for confirmation before writing

**Quality standard:** write pattern bullets for a future AI instance, not a human reader.
Be precise and transferable. Bad: "Tom likes clear writing." Good: "Leads with the key point,
follows with supporting detail — never buries the lede."

---

## Writing Updates

Once confirmed:

1. Use `Notion:notion-update-page` with `command: update_content` to add the new bullet
   to the correct subheading under `## Current Patterns`
2. Update `## Last Reviewed` to today's ISO date
3. Append a row to the `## Changelog` table:
   `| YYYY-MM-DD | [short description of what was added or changed] | [session context] |`

Do not rewrite existing bullets unless the new evidence directly contradicts them. If
contradicted, flag the conflict and propose revised wording rather than silently overwriting.

---

## Periodic Review (Bootstrap Integration)

When this skill runs as part of the bootstrap workflow at session start:

1. Fetch the tone page
2. Check **Last Reviewed** — if more than 30 days ago, note: "Tone page last reviewed
   [date] — worth scanning this session for new patterns."
3. Surface any **Open Questions** that the current conversation might resolve
4. Otherwise, load patterns silently and continue

---

## Updating This Skill

This skill file cannot be updated automatically. Whenever a change is agreed, Claude must
produce the complete updated SKILL.md as a downloadable file and tell Tom to replace the
current version in his skills directory. Accumulate all changes across a conversation and
produce one consolidated file at the end.
