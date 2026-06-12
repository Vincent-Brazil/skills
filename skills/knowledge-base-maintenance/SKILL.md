---
name: knowledge-base-maintenance
description: >
  Manages a persistent Notion knowledge base that stores solutions, decisions, and context
  across conversations. Use at the START of any conversation that references past work,
  ongoing projects, or anything where prior context would change the response. Use at the
  END of any conversation where a solution, decision, workaround, or useful approach was
  reached. If in doubt, use this skill — it is better to check and find nothing relevant
  than to miss capturing something useful. Triggers include: "do you remember", "last time
  we", "continue from", "add this to the knowledge base", "update the KB", or any phrasing
  that implies ongoing work with shared history.
compatibility: "Requires Notion MCP connection for reading and updating pages."
---

# Knowledge Base Maintenance Skill

Tom maintains a Notion knowledge base as persistent memory and handoff context for Claude
across conversations. This skill governs when and how to read it, write to it, and keep it healthy.

---

## Updating This Skill

This skill file cannot be updated automatically. Whenever a change is agreed, Claude must
produce the complete updated SKILL.md as a downloadable file and tell Tom to replace the
current version in his skills directory. Do NOT claim the skill has been updated unless a
file has been produced. Accumulate all changes across a conversation and produce one
consolidated file at the end.

---

## Core Notion Pages

- **Main knowledge base**: ID `35775631-9aa7-815f-ab40-fb0e6d4bf033`

Knowledge area pages (fetch on demand):
- **Jira & JQL**: ID `35c75631-9aa7-817c-8a12-cac5afbdc675` `[active]`
- **Local Jira Reporting Tools**: ID `35775631-9aa7-81da-8218-e741ec61c5ca` `[active]`
- **Internal AI & Adoption**: ID `35c75631-9aa7-81c6-8478-d47d7605d8d4` `[dormant]`
- **AI Architecture**: ID `358756319aa781cc8ef4c0402dc74494` `[active]`
- **Career & Job Applications**: ID `35a75631-9aa7-81a9-8b45-e14ba2294644` `[reference]`
- **Gaming & Discord**: ID `35c75631-9aa7-81e0-a96d-c20c420acb28` `[reference]`

Active project pages (IDs stored on main KB navigation page — fetch from there):
- **Sprint Reporter** `[active — scope & design]`
- **Strategic Alignment Report** `[active — scope & design]`
- **First Vibe-Coded Module** `[reference]`
- **Programme Relationship Graph** `[active — vision]`
- **PRG PoC Build Plan** `[active — scope & design]`
- **PRG Delivery Log** `[active — current state & tasks]`
- **ViewQueue** `[active — vision]`
- **ViewQueue Scope & Design** `[active — scope & design]`
- **ViewQueue Delivery Log** `[active — current state & tasks]`

New sub-pages should be created as child pages of the main KB page (parent ID
`35775631-9aa7-815f-ab40-fb0e6d4bf033`), then linked from the Knowledge Areas section.

---

## Page Types and Labels

Every KB page carries a short label signalling its trustworthiness and role.

| Label | Meaning |
|---|---|
| `[active — vision]` | Project vision. Trustworthy. Fetch for orientation and key decisions. |
| `[active — scope & design]` | Project architecture. Trustworthy. Fetch when discussing how something is built. |
| `[active — current state & tasks]` | Project delivery. High staleness risk — updated every session. Fetch first when picking up a project. |
| `[active]` | Single-page knowledge area. Trustworthy. |
| `[dormant]` | Not recently updated. Verify before acting on it. |
| `[reference]` | Templates, rules, specs, logs. Fetch relevant sections on demand. |

### Three-page structure for active projects

Pages are created as a project matures — do not pre-create speculatively.

- **Vision** `[active — vision]` — the why and what. Changes only when direction changes.
- **Scope & Design** `[active — scope & design]` — the how. Create when the approach is defined.
- **Current State & Tasks** `[active — current state & tasks]` — the where. Create when active work begins. Update every session.

If a conversation moves into scope definition and no Scope & Design page exists, propose creating one. If work has clearly started and there is no Current State & Tasks page, propose creating one.

### Staleness risk

- `[active — current state & tasks]`: high. Flag any mismatch with what is being discussed.
- `[active]` knowledge area pages: medium. Flag if Current State looks out of date.
- `[active — vision]` / `[active — scope & design]`: low. Flag only if something contradicts the conversation.
- `[dormant]`: treat as potentially wrong. Verify before acting.
- `[reference]`: low for stable material; higher for logs and trackers.

---

## Reading the Knowledge Base

The main KB page is always the entry point. Sub-pages are fetched when there is reasonable overlap with the conversation — default toward fetching rather than skipping. The cost of an extra Notion call is low; the cost of missing context is high.

Once the main KB is fetched, use it to identify which sub-pages are relevant. Fetch sub-pages when the main KB summary isn't enough detail — for specific decisions, config, scripts, queries, or current project state.

For active projects, fetch Vision first when starting cold or when direction, rationale, or scope is relevant — it carries the orientation layer that stops Claude going off course. Then fetch Current State & Tasks for where things stand. Fetch Scope & Design on top if architectural detail is needed.

For knowledge area pages, fetch the relevant sub-page whenever the topic has reasonable overlap. If in doubt, fetch it.

**How to read:**
1. Fetch the main KB using `Notion:notion-fetch` with the main KB ID
2. Check Open Threads — surface any relevant ones naturally at the start of the conversation
3. Fetch relevant sub-pages
4. Use the context without announcing you have read it

### Sub-page structure for knowledge area pages

- **## What This Is** — brief framing of the topic
- **## Current State** — where things stand; most important for cold pickup
- **## Key Decisions & Rationale** — the why; what was tried and failed
- **## Tasks** — ongoing and backlog items (omit if no active work)
- **## Open Questions / Next Steps** — topic-specific loose ends
- **## History / Background** — older context, not current state

Omit sections that do not apply rather than leaving them empty.

---

## Open Threads

Open Threads on the main KB page captures directions to explore, loose ends, and things to pick up next time. It is distinct from project Tasks, which are work items on individual project pages.

At the end of any conversation where an interesting tangent emerged or Tom expressed interest in returning to something, propose: "Worth adding to Open Threads? [description]"

Format: `- **[date]** [Short title] — [one sentence on what it is and why it matters]`

Once confirmed, add to Open Threads using `Notion:notion-update-page` with `command: update_content`. When a thread is resolved in a later conversation, remove it and add a proper KB entry if warranted.

---

## Writing KB Entries

Propose an entry at the end of any conversation where a working solution was found, a decision was made and the reasoning matters, a workaround was identified, a preference was clarified, or a new tool or page was set up with IDs that will be needed again.

Do not propose for purely exploratory conversations with no conclusion, or information already in the KB.

**Quality standard:** write for a future AI instance, not a human reader. The goal is to transfer understanding — context, reasoning, and what was tried — not just conclusions. Be specific: include IDs, file names, config values, exact terminology. Flag anything a future Claude might get wrong.

**Placement:** propose an existing sub-page section if it clearly extends something there. Propose a new sub-page if the topic has ongoing status, config details, or active tasks, or would exceed ~20 lines. Main page only for Comms & Tone, General Preferences, or navigation content.

**Confirming and writing:**
1. Present the proposed entry and placement; wait for confirmation
2. Write using `Notion:notion-update-page` with `command: update_content`
3. Update the **Recently Added** section of the main KB with a one-line summary and date
4. For new sub-pages, use `Notion:notion-create-pages` with the main KB as parent, then link from Knowledge Areas

---

## Keeping the KB Healthy

Watch for and flag:
- **Stale Open Threads** — resolved threads not yet removed
- **Outdated Current State** — pages that look out of date relative to the conversation
- **Missing project pages** — a project with a Vision but no Scope & Design or Current State & Tasks as work progresses
- **Dormant pages used as active** — flag and offer to verify
- **Duplication** — similar content in multiple places; offer to consolidate
- **Log file suggestions** — Git is the change log for repos; do not suggest log.md equivalents
