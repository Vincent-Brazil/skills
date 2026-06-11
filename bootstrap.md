# Bootstrap v1.1

## How to Use This File

This file is the skill index for this Claude instance. At the start of every session,
fetch this file from GitHub and hold it in context for the session:

    https://raw.githubusercontent.com/Vincent-Brazil/skills/main/bootstrap.md

When a skill appears relevant during a conversation, fetch and apply it from the URL
listed below. Do not fetch skills proactively — only load them when the conversation
warrants it.

At the point any skill is first fetched, also silently check whether the repo bootstrap
is ahead of the version loaded at session start. Do this by fetching the URL above and
comparing the version number in the header. If the repo version is higher, notify:

> ⚠️ Bootstrap is out of date (loaded: v1.0, repo: vX.X). Please copy the latest
> bootstrap.md from GitHub into your Claude Project instructions to stay in sync.

Then continue the session normally using the already-fetched skill content.

---

## Skills

| Skill | When to invoke | URL |
|---|---|---|
| knowledge-base-maintenance | At the START of any conversation referencing past work, ongoing projects, or prior context. At the END of any conversation where a solution, decision, workaround, or useful approach was reached. When user says "do you remember", "last time we", "add to KB", or implies shared history. Requires Notion access (see platform note). | https://raw.githubusercontent.com/Vincent-Brazil/skills/main/skills/knowledge-base-maintenance.md |

**Platform note:** skills are written against claude.ai connector tools (e.g. the Notion
MCP for knowledge-base-maintenance). On instances without the connector — such as CLI
environments like Claude Code — substitute the local equivalents defined in that
instance's private instructions. Per-platform wiring is documented privately in the
knowledge base ("AI Instance Setup" page), not in this public repo.

---

## Adding or Updating Skills

When a new skill is added to the repo or an existing one is updated:

1. Add or update the entry in the Skills table above
2. Increment the version number in the `# Bootstrap` header
3. Commit both files to the repo

The next session on any Claude instance will detect the version bump and prompt the user
to update their local Project instructions.
