# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`idobata` is a Claude Code plugin that ships **議論プロトコル群** as skills. Each skill orchestrates a multi-agent debate format (Six Thinking Hats / Hegelian / Red-Blue / Pugh Matrix / Rational Choice / Multi-Agent Debate / Delphi / SCAMPER / Morphological Analysis / Brainstorm / Three-Hats), with a `meta-router` skill `/idobata:debate` that auto-selects the best protocol for a given topic.

There is no application code, no build, no tests in the conventional sense. The repo is pure markdown: skill definitions, subagent definitions, and a plugin manifest. Edits ship by re-running `/reload-plugins`.

## Hard prerequisite

All skills depend on the **Agent Teams** experimental Claude Code feature. `.claude/settings.json` already has:
```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```
Without this, `Agent` / `TeamCreate` tools are unavailable and every skill fails on the first spawn.

## Architecture

### Two parallel installations of the same content

- `plugin/` — canonical plugin source, referenced by `.claude-plugin/marketplace.json` (`source: ./plugin`). This is what gets distributed.
- `.claude/skills/idobata-*/`, `.claude/agents/idobata-*.md` — **partial subset** copied locally for direct testing without going through the marketplace install. Only 4 skills (debate / brainstorm / scamper / morphological) and 4 agents are mirrored here. The full set lives only in `plugin/`.

When editing a skill, edit `plugin/skills/<name>/SKILL.md`. If the same name exists under `.claude/skills/idobata-<name>/`, mirror the change there too (otherwise local invocation drifts from plugin invocation).

### Skill ↔ subagent contract (the big picture)

Every skill follows the same architectural pattern. Reading 1 SKILL.md without this map is misleading.

1. **Main session = lead**. Each SKILL.md tells the model "you are lead / synthesizer / judge / blue hat / facilitator". The lead is *not* spawned — it's the model running the user's slash command.
2. **Lead spawns N subagents in parallel** via the `Agent` tool, with `subagent_type` referencing a definition in `plugin/agents/<name>.md` by its frontmatter `name` field.
3. **Subagents are role-constrained**: each agent definition forbids the agent from doing anything outside its assigned lens / role. They return structured output in a fixed format the lead aggregates.
4. **Lead runs protocol rounds**: each SKILL.md defines its own round structure (Round 0 framing → Round 1 parallel → Round 2 yes-and / refinement → final synthesis). This is custom procedural logic baked into the prompt; Agent Teams provides only the spawning primitive.
5. **Output format is mandated** by the SKILL.md and reproduced verbatim by the lead at the end (cluster summaries, pugh matrix tables, comparison reports, etc.).

`debate` is the only skill that does not orchestrate subagents directly — instead the lead inspects the topic, picks the best protocol via a routing table, and invokes that skill via the `Skill` tool with the original arguments forwarded.

### Agent role mapping (plugin/agents/ → owning skill)

Most agents are skill-specific. Mapping is documented in README.md but not enforced; the same name field in frontmatter is what `subagent_type` resolves to. When adding a new protocol, place its agents in `plugin/agents/` with descriptive `name:` values and reference them from the new SKILL.md.

## Editing & reloading

- After any edit under `plugin/skills/` or `plugin/agents/`, run `/reload-plugins` in the user's session for the changes to take effect.
- `.claude/skills/` and `.claude/agents/` files are picked up directly without `/reload-plugins`, but a session restart is still needed for skill registry updates.

## Authoring conventions worth knowing before editing

- **Generality over specificity**: skills are meant to apply to any domain. Avoid examples that pin a skill to a single use case (e.g. feature engineering, finance) — the project history has been deliberately scrubbed of those. Use neutral examples (Web service ideas, event design, naming) that any reader can map to their own domain.
- **Divergence skills must not include critic roles**: `brainstorm` / `scamper` / `morphological` deliberately omit critique. Don't add "evaluate the ideas" steps — that pushes the lead into convergence and kills idea volume. Convergence is delegated to a follow-up skill (`pugh-matrix`, `rational-choice`, `red-blue`).
- **Skill names use slash form `/idobata:foo`**, but `subagent_type` references use the plain `name:` from the agent's frontmatter (no `idobata:` prefix). Mixing these breaks spawning.
- The `description:` frontmatter on each SKILL.md is what determines auto-firing from natural-language input. As of 2026-05, empirical testing showed the meta-router's recall on natural language was 0% (Claude opus answers directly without invoking the skill). This repo is therefore designed around **explicit invocation** (`/idobata:foo <topic>`); auto-firing is not a goal.

## Marketplace source

`.claude-plugin/marketplace.json` uses `"source": "./plugin"` — pure local reference. To distribute via GitHub, change to `"source": "github:owner/repo"`. The marketplace name is `idobata-local`; the plugin name is `idobata`.

## Known leftover state

- `.claude/commands/` may exist as an empty directory. It's a legacy artifact from before skills-format migration and is sometimes recreated by external eval tooling (skill-creator's `run_eval.py` writes temp commands there). Safe to delete; do not put new content there.
