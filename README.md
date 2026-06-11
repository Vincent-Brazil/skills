# AI Skills

A shared skill library for AI assistant instances — Claude.ai Projects, Claude Code, and
other agents. One set of skills, many instances: each instance bootstraps from this repo
so behaviour stays consistent everywhere.

## How it works

- **[bootstrap.md](bootstrap.md)** — the versioned skill index and entry point. Instances
  fetch it at session start, hold it in context, and load individual skills on demand.
- **[skills/](skills/)** — individual skill files with YAML frontmatter (`name`,
  `description`). The description tells the instance when to invoke the skill.
- **Versioning** — the bootstrap header carries a version number. When an instance fetches
  a skill, it re-checks the repo bootstrap; if the repo version is ahead of the copy in the
  instance's local instructions, it prompts the user to re-sync.

## Connecting an instance

### Claude.ai (Projects)

1. Create a Claude Project.
2. Copy the full contents of `bootstrap.md` into the Project instructions.
3. Connect any connectors the skills require (e.g. the Notion connector for
   knowledge-base-maintenance) and grant them access to the relevant workspace pages.
4. Done. Skills are fetched on demand, and the version check warns when the pasted
   bootstrap goes stale.

### Claude Code

1. Add a session-start instruction to `~/.claude/CLAUDE.md`:

   ```
   At the start of every session, fetch and load:
   https://raw.githubusercontent.com/Vincent-Brazil/skills/main/bootstrap.md
   Follow its instructions: hold it in context, load listed skills on demand,
   and honour its version-check rule.
   ```

2. Optionally install frequently used skills locally under
   `~/.claude/skills/<skill-name>/SKILL.md` so they auto-trigger without a fetch. Locally
   installed copies must be re-synced manually when the repo version changes.
3. Claude Code has no claude.ai connectors. Where a skill references connector tools
   (e.g. Notion MCP), map them in `~/.claude/CLAUDE.md` to local equivalents — for
   example a CLI helper script — so the skill's process applies unchanged.

### Other agents

Any agent that can fetch a URL can use this repo: load `bootstrap.md` at session start
and follow it. If the agent lacks the tools a skill assumes, substitute local equivalents
or treat the skill as read-only guidance.

## Keeping personal data out

This repo is public. Skills describe **process**, written generically. Environment- and
person-specific details — workspace page IDs, local file paths, machine names,
credentials — belong in each instance's private instruction layer (Project instructions,
`CLAUDE.md`, private notes), never in this repo.

## Adding skills

See the "Adding or Updating Skills" section in [bootstrap.md](bootstrap.md): add the
skill file under `skills/`, register it in the bootstrap's table, and bump the bootstrap
version.
