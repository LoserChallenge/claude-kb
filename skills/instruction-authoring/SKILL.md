---
name: instruction-authoring
description: Routes an instruction to the right Claude Code construct — CLAUDE.md, .claude/rules/, skills, subagents, hooks, settings.json, permissions, MCP, or plugins — and covers Anthropic's published guidance on phrasing instructions Claude follows reliably. Use when deciding where an instruction belongs, when writing or revising a CLAUDE.md, SKILL.md, subagent definition, or hook, when an instruction is being ignored or a skill triggers unreliably, or when mapping a Claude Code setup onto claude.ai projects, profile preferences, and styles.
---

# Instruction authoring for Claude Code

Two separate questions. Diagnose which one you are answering before reaching for the reference file.

- **Placement** — Claude never mentions the rule, or it disappears after compaction.
- **Phrasing** — Claude sees the rule and follows it inconsistently.

## Route a placement question here first

Ask in order, stop at the first `yes`.

1. Must this hold **every time**, regardless of what Claude decides? → **Hook**. An instruction in CLAUDE.md or a skill is a request, not a guarantee.
2. Is this a **hard capability boundary** — a tool, path, or command Claude must never touch? → **`permissions.deny`**. Permission rules are enforced by Claude Code, not by the model.
3. Needed in **every session**, and short? → **CLAUDE.md** (or `.claude/rules/` if it is one topic among many). Target under 200 lines.
4. Needed **only sometimes** — reference material or a triggered procedure? → **Skill**.
5. Changes Claude's **role, tone, or default response format** every turn? → **Output style**.

Otherwise: scoped to a file type → a rule with `paths:` frontmatter. Floods context with output → a subagent. Reaches an external system → an MCP server. True only right now → say it in the prompt.

## Read the reference file for anything beyond that

`reference.md` in this skill directory. Do not read it whole — open the section you need.

| Question | Section |
|:---|:---|
| Full precedence chain for CLAUDE.md, rules, local, managed | 3 |
| Do `@`-imports save context? (No) | 4 |
| Skill triggering, description budgets, invocation control | 6, 20 |
| What a subagent actually receives at startup | 7 |
| Hook locations, merge behavior, what hooks can and cannot enforce | 8 |
| Permission evaluation order and settings precedence | 9 |
| Precedence for every construct, side by side | 12 |
| What survives `/compact` | 11 |
| Before/after pairs for vague instructions | 13 |
| Positive vs. negative framing | 14 |
| Whether to add "IMPORTANT" or "YOU MUST" — **Anthropic's sources conflict** | 15 |
| Length targets per artifact type | 19 |
| Writing a description that triggers correctly | 20 |
| Verifying a change actually worked | 23 |
| claude.ai equivalents, and what has none | 24, 25 |

## Standing constraints

- Every claim in `reference.md` carries a source tag. When quoting it, carry the tag.
- Claims marked `[UNVERIFIED]` are not evidence. Say so rather than passing them on as fact.
- The file was compiled 2026-08-02 against docs that change weekly. For anything version-sensitive, check Section 26 and re-verify against https://code.claude.com/docs/en/claude_code_docs_map.md before relying on it.
