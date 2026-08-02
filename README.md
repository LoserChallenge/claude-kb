# claude-kb

Shared reference library for Claude Code and Claude web. One topic per file, all under `docs/`.

**Do not answer from memory about what these docs contain — open the file.**

**Start here — this file is the entry point.** There is no raw URL for a repo or a folder; raw
URLs resolve to single files. So the way in is to fetch this index, then fetch the doc it names.

| | |
|:---|:---|
| From Claude web | fetch `https://raw.githubusercontent.com/LoserChallenge/claude-kb/main/README.md`, then the doc's raw link |
| From Claude Code | invoke the `claude-kb` skill, or `/claude-kb` |
| Local clone | `D:\cc-tech-support\claude-kb` |
| Raw URL pattern | `https://raw.githubusercontent.com/LoserChallenge/claude-kb/main/<path>` |
| If a doc is missing from this index | list the folder: `https://api.github.com/repos/LoserChallenge/claude-kb/contents/docs` — each entry carries a `download_url` |

---

## Docs

### instruction-authoring

`docs/instruction-authoring.md` — [raw](https://raw.githubusercontent.com/LoserChallenge/claude-kb/main/docs/instruction-authoring.md)

Where a Claude Code instruction belongs — CLAUDE.md, `.claude/rules/`, skills, subagents, hooks,
`settings.json`, permissions, MCP, plugins — and how to phrase it so Claude follows it reliably.
Also maps Claude Code constructs onto claude.ai projects, profile preferences, and styles.

Two separate questions; diagnose which one before reading. **Placement** — Claude never mentions
the rule, or it disappears after compaction. **Phrasing** — Claude sees the rule and follows it
inconsistently.

Compiled 2026-08-02 against Anthropic sources only. Re-verify version-sensitive claims after
roughly 2026-11-01.

**Jump table** — open the section, not the whole file:

| Question | Section |
|:---|:---|
| Route one instruction to the right construct, fast | 1 |
| Token cost as the deciding constraint | 2 |
| Full precedence chain for CLAUDE.md, rules, local, managed | 3 |
| Do `@`-imports save context? (No) | 4 |
| Skill triggering, description budgets, invocation control | 6, 20 |
| What a subagent actually receives at startup | 7 |
| Hook locations, merge behavior, what hooks can and cannot enforce | 8 |
| Permission evaluation order and settings precedence | 9 |
| What survives `/compact` | 11 |
| Precedence for every construct, side by side | 12 |
| Before/after pairs for vague instructions | 13 |
| Positive vs. negative framing | 14 |
| Whether to add "IMPORTANT" or "YOU MUST" — **Anthropic's sources conflict** | 15 |
| Length targets per artifact type | 19 |
| Writing a description that triggers correctly | 20 |
| Verifying a change actually worked | 23 |
| claude.ai equivalents, and what has none | 24, 25 |
| Version floors and surface-specific caveats | 26 |

---

## Adding a doc

1. Put the file in `docs/`.
2. Add a section here: path, raw link, what it covers, compile date, jump table.
3. Commit and push.

The `claude-kb` skill reads this index at run time, so a new doc is picked up without editing the
skill — unless its subject falls outside what the skill's `description` names, in which case add
the subject there too.

## How these docs are meant to be read

Every doc is a compiled snapshot, not live truth.

- Claims carry source tags. Carry the tag when you quote one.
- `[UNVERIFIED]` is not evidence — say so rather than passing it on as fact.
- Check the compile date before relying on anything version-sensitive.
