# Bootstrap v1.6

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

> ⚠️ Bootstrap is out of date (loaded: vX.X, repo: vY.Y). Please copy the latest
> bootstrap.md from GitHub into your Claude Project instructions to stay in sync.

Then continue the session normally using the already-fetched skill content.

---

## Skills

| Skill | When to invoke | URL |
|---|---|---|
| knowledge-base-maintenance | At the START of any conversation referencing past work, ongoing projects, or prior context. At the END of any conversation where a solution, decision, workaround, or useful approach was reached. When user says "do you remember", "last time we", "add to KB", or implies shared history. Requires Notion access (see platform note). | https://raw.githubusercontent.com/Vincent-Brazil/skills/main/skills/knowledge-base-maintenance/SKILL.md |
| poc-decomposition | When working on a PoC that uses the three-level Vision → Phase → Task architecture: to validate a phase, decompose it into tasks, update the task ledger after execution, aggregate results, and determine phase transition readiness. | https://raw.githubusercontent.com/Vincent-Brazil/skills/main/skills/poc-decomposition/SKILL.md |
| number-one | ONLY when the user explicitly invokes it ("/number-one", "engage Number One mode", "talk like a Star Trek first officer"). Never trigger proactively — it is a fun session persona (Starfleet XO addressing the user as Captain), and unsolicited activation would be annoying. The `disable-model-invocation` frontmatter flag enforces this on Claude Code; other instances must honour this rule manually. | https://raw.githubusercontent.com/Vincent-Brazil/skills/main/skills/number-one/SKILL.md |
| tone-voice | At the END of any session where a new phrasing habit, style preference, or voice quirk emerged. At the START of any session involving writing, editing, or style-matching to Tom's voice. Triggers: "does that sound like me", "write it in my voice", "capture how I phrase things", "update my tone profile". Requires Notion access. | https://raw.githubusercontent.com/Vincent-Brazil/skills/main/skills/tone-voice/SKILL.md |

**Platform note:** skills are written against claude.ai connector tools (e.g. the Notion
MCP for knowledge-base-maintenance). On instances without the connector — such as CLI
environments like Claude Code — substitute the local equivalents defined in that
instance's private instructions. Per-platform wiring is documented privately in the
knowledge base ("AI Instance Setup" page), not in this public repo.

---

## Adding or Updating Skills

Skills follow the Agent Skills open standard layout: one folder per skill containing a
`SKILL.md` with YAML frontmatter (`name`, `description`) and markdown instructions —
`skills/<skill-name>/SKILL.md`. This lets tools that support the standard natively
(Claude Code, Codex CLI, Gemini CLI, and others) load skills directly, while this
bootstrap serves instances that fetch over HTTP.

When a new skill is added to the repo or an existing one is updated:

1. Create or update `skills/<skill-name>/SKILL.md`
2. Add or update the entry in the Skills table above
3. Increment the version number in the `# Bootstrap` header
4. Commit both files to the repo

The next session on any Claude instance will detect the version bump and prompt the user
to update their local Project instructions.
