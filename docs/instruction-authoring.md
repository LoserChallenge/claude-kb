# Writing Instructions for Claude Code: Placement and Phrasing

**Compiled:** 2026-08-02. Every substantive claim below was verified against a source fetched on that date. Re-verify before trusting this file after roughly 2026-11-01; Claude Code ships weekly and several claims here already carry version floors.

**How to read the source tags.** Every claim carries a bracketed tag pointing at the source key below. Claims that could not be verified against a page fetched during compilation are marked `[UNVERIFIED]` inline. Unmarked, untagged text is the compiling session's organization of tagged facts — routing procedures, decision tables, and reconciliations — not new factual claims.

**One deliberate exception to a rule this file states.** Section 22 reproduces Anthropic's guidance to avoid time-sensitive information, and this file is dated and version-pinned throughout. That guidance targets *skills*, which load into live sessions; this is a reference artifact whose value depends on a later session being able to judge what has gone stale. Version pins are quarantined in Section 26 rather than scattered through the placement and phrasing sections, so the durable guidance stays readable as the pins age.

## Source key

| Tag | URL | Tier |
|:---|:---|:---|
| `[features]` | https://code.claude.com/docs/en/features-overview.md | 1 — docs |
| `[memory]` | https://code.claude.com/docs/en/memory.md | 1 — docs |
| `[skills]` | https://code.claude.com/docs/en/skills.md | 1 — docs |
| `[subagents]` | https://code.claude.com/docs/en/sub-agents.md | 1 — docs |
| `[hooks-guide]` | https://code.claude.com/docs/en/hooks-guide.md | 1 — docs |
| `[settings]` | https://code.claude.com/docs/en/settings.md | 1 — docs |
| `[permissions]` | https://code.claude.com/docs/en/permissions.md | 1 — docs |
| `[output-styles]` | https://code.claude.com/docs/en/output-styles.md | 1 — docs |
| `[mcp]` | https://code.claude.com/docs/en/mcp.md | 1 — docs |
| `[plugins]` | https://code.claude.com/docs/en/plugins.md | 1 — docs |
| `[context]` | https://code.claude.com/docs/en/context-window.md | 1 — docs |
| `[cc-best]` | https://code.claude.com/docs/en/best-practices.md | 1 — docs |
| `[skill-bp]` | https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices | 1 — docs |
| `[prompt-bp]` | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices | 1 — docs |
| `[sup-projects]` | https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects | 2 — support |
| `[sup-personalization]` | https://support.claude.com/en/articles/10185728-understanding-claude-s-personalization-features | 2 — support |

No practitioner or community sources were used. Where Anthropic publishes nothing on a point, this file says so rather than filling the gap.

## Contents

**Part One — Placement** (which construct holds this instruction)
1. Route a single instruction without reading the rest of this file
2. Token cost is the constraint that decides most placements
3. CLAUDE.md and its full precedence chain — including `.claude/rules/`
4. `@`-imports and how memory files compose
5. Output styles
6. Skills — and note that custom slash commands are now skills
7. Subagents — separate context, separate instructions
8. Hooks — the only deterministic layer for behavior
9. settings.json and permissions
10. Plugins and MCP configuration
11. Session-level steering — plan mode, `/compact`, `/clear`, compaction survival
12. Precedence at a glance

**Part Two — Phrasing** (how to word it so Claude follows it)
13. Specificity — the highest-yield lever
14. Positive versus negative framing
15. Emphasis and its failure modes — **sources conflict; read before adding "IMPORTANT"**
16. Give rationale
17. Examples
18. Structure and headers
19. Length
20. Writing descriptions that trigger correctly
21. Match specificity to the task's fragility
22. Consistency, terminology, and anti-patterns
23. Verify a phrasing change actually worked

**Part Three — Mapping to claude.ai**
24. Which Claude Code construct maps to which claude.ai feature
25. Where no claude.ai equivalent exists
26. Version-specific and surface-specific guidance

**Part Four — Self-check**
27. Before writing or revising any instruction file
28. Two queries that should produce different behavior if this file is loaded

---

# PART ONE — PLACEMENT

## 1. Route a single instruction without reading the rest of this file

Given one instruction, ask these five questions in order and stop at the first `yes`.

| # | Ask | If yes → | Why |
|:---|:---|:---|:---|
| 1 | Must this hold **every time**, regardless of what Claude decides? | **Hook** (`PreToolUse` to block, `PostToolUse` to react) | An instruction in CLAUDE.md or a skill is a request, not a guarantee; a hook that blocks the edit is enforcement `[features]` |
| 2 | Is this a **hard capability boundary** — a tool, path, or command Claude must never touch? | **`permissions.deny` in settings.json** | Permission rules are enforced by Claude Code, not by the model; prompt and CLAUDE.md text shape what Claude tries but don't change what is allowed `[permissions]` |
| 3 | Does Claude need to know this in **every session**, and is it short? | **CLAUDE.md** (project root, or `.claude/rules/` if it's one topic among many) | Loads in full at session start, costs tokens on every request `[features]` |
| 4 | Is this needed **only sometimes** — reference material, or a procedure with a trigger? | **Skill** (`.claude/skills/<name>/SKILL.md`) | Body loads only when used, so long reference material costs almost nothing until needed `[skills]` |
| 5 | Does this change Claude's **role, tone, or default response format** on every turn, independent of the project? | **Output style** | Modifies the system prompt itself; everything else adds context around it `[output-styles]` |

If none matched, it is one of these:

- **Scoped to a file type or directory** → a rule in `.claude/rules/` with `paths:` frontmatter, which loads only when Claude touches matching files `[memory]`
- **A side task that would flood the conversation with output** → a subagent `[features]`
- **Access to an external system** → an MCP server `[mcp]`
- **True for this repo but personal to you** → `CLAUDE.local.md`, gitignored `[memory]`
- **Only true for this one task, right now** → say it in the prompt, or use plan mode; do not persist it

**The two most common misroutings, both correctable at a glance:**

1. A rule that must hold every time was written into CLAUDE.md. Anthropic states this directly: *"An instruction like 'never edit `.env`' in CLAUDE.md or a skill is a request, not a guarantee. A `PreToolUse` hook that blocks the edit is enforcement. If a rule must hold every time, make it a hook rather than a prompt instruction."* `[features]`
2. Reference material was written into CLAUDE.md. Move it to a skill. Anthropic's stated rule of thumb: keep CLAUDE.md under 200 lines; if it's growing, move reference content to skills or split into `.claude/rules/` `[features]`.

## 2. Token cost is the constraint that decides most placements

Treat this table as the reason behind Section 1, and consult it whenever a placement is contested `[features]`:

| Construct | When it loads | What loads | Context cost |
|:---|:---|:---|:---|
| CLAUDE.md | Session start | Full content of all levels | **Every request** |
| Skills | Session start + when used | Descriptions at start, full body when used | Low; zero for `disable-model-invocation: true` |
| MCP servers | Session start | Tool names; full schemas deferred | Low until a tool is used |
| Subagents | When spawned | Fresh isolated context | Isolated from the main session |
| Hooks | On trigger | Nothing (runs externally) | **Zero**, unless the hook returns output |
| Output styles | Session start | Appended to system prompt | Every request, but prompt-cached after the first `[output-styles]` |

Directives that follow from this table:

- **Never put anything in CLAUDE.md that Claude only needs sometimes.** CLAUDE.md is the only construct whose full content is billed on every single request `[features]`.
- **Prefer `disable-model-invocation: true` on any skill with side effects.** It removes the description from context entirely, taking the skill's cost to zero until manually invoked, and prevents Claude deciding to deploy on its own `[skills]`.
- **When a rule only matters for some files, add `paths:` frontmatter.** Path-scoped rules load only when Claude reads a matching file `[memory]`.
- **Route high-volume work to a subagent.** The subagent's file reads never touch the main context; only its summary returns `[features]`.

## 3. CLAUDE.md and its full precedence chain

**What it is for.** Facts Claude should hold in *every* session: build commands, conventions, project layout, "always do X" rules `[memory]`.

**When it loads.** At session start, in full, for every file in the hierarchy above the working directory. Subdirectory files load on demand when Claude reads a file in that directory `[memory]`.

**Critical mechanical detail that changes how you write it:** CLAUDE.md content is delivered **as a user message after the system prompt, not as part of the system prompt itself**. Claude reads it and tries to follow it, but there is no guarantee of strict compliance, especially for vague or conflicting instructions `[memory]`. Output styles and `--append-system-prompt` are the only constructs that reach the system prompt `[output-styles]`.

### The chain, in load order (broadest scope first) `[memory]`

| Scope | Location | Use for |
|:---|:---|:---|
| **Managed policy** | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`; Linux/WSL `/etc/claude-code/CLAUDE.md`; Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | Org-wide coding standards, security policy, compliance |
| **User** | `~/.claude/CLAUDE.md` | Personal preferences across all projects |
| **Project** | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team-shared, committed to source control |
| **Local** | `./CLAUDE.local.md` | Personal project-specific; add to `.gitignore` |

**Precedence is additive, not override.** All discovered files are concatenated into context rather than overriding each other. Across the tree, content is ordered filesystem-root-down to the working directory, so instructions closer to where Claude launched are read last. Within each directory, `CLAUDE.local.md` is appended after `CLAUDE.md` `[memory]`. When instructions conflict, **Claude uses judgment to reconcile them, with more specific instructions typically taking precedence** — this is explicitly a judgment call, not a deterministic rule `[features]`.

**Directive:** because conflicts resolve by judgment rather than by rule, do not rely on a lower-level file to "override" a higher one. Remove the contradiction instead. Anthropic states that if two rules contradict, Claude may pick one arbitrarily `[memory]`.

**Managed policy specifics:**
- Managed policy CLAUDE.md **cannot be excluded** by any individual setting `[memory]`.
- The `claudeMd` key in `managed-settings.json` injects CLAUDE.md content directly, with the same precedence as a managed file. Honored **only** in managed/policy settings; setting it in user, project, or local settings has no effect `[memory]`.
- Anthropic draws the enforcement line explicitly: use **managed settings** for blocking tools/commands/paths, sandbox enforcement, env vars, and auth locks; use **managed CLAUDE.md** for code style, data-handling reminders, and behavioral instructions. *"Settings rules are enforced by the client regardless of what Claude decides to do. CLAUDE.md instructions shape Claude's behavior but are not a hard enforcement layer."* `[memory]`

**Monorepo escape hatch:** `claudeMdExcludes` accepts globs or absolute paths and skips matching CLAUDE.md files. Configurable at any settings layer; arrays merge across layers. Cannot exclude managed policy files `[memory]`.

**AGENTS.md:** Claude Code reads `CLAUDE.md`, not `AGENTS.md`. If a repo already uses `AGENTS.md`, create a `CLAUDE.md` that imports it with `@AGENTS.md`, then append Claude-specific instructions below `[memory]`.

### `.claude/rules/` — the pressure valve for a growing CLAUDE.md

Place topic-scoped markdown files in `.claude/rules/`; all `.md` files are discovered recursively `[memory]`.

- Rules **without** `paths:` frontmatter load at launch with the **same priority as `.claude/CLAUDE.md`** `[memory]`.
- Rules **with** `paths:` frontmatter load only when Claude reads a matching file — this is the main context-saving move for large repos `[memory]`.
- User-level rules live in `~/.claude/rules/` and load **before** project rules, giving project rules higher priority `[memory]`.
- Path patterns use globs; brace expansion is bounded at 1,000 expanded patterns and 4 MiB per rule `[memory]`.

**Directive:** use rules when CLAUDE.md is growing but the content still needs to be present every session or on every touch of a file type. Use a **skill** instead when the content is a procedure or reference material Claude needs only sometimes — Anthropic's own note: *"For task-specific instructions that don't need to be in context all the time, use skills instead"* `[memory]`.

## 4. `@`-imports and how memory files compose

**Syntax:** `@path/to/import` anywhere in a CLAUDE.md file `[memory]`.

**The placement-relevant fact:** imported files are **expanded and loaded into context at launch**, alongside the CLAUDE.md that references them. Splitting into imports helps organization but **does not reduce context** `[memory]`. Anthropic repeats this in the troubleshooting section: *"Splitting into `@path` imports helps organization but doesn't reduce context, since imported files load at launch"* `[memory]`.

**Directive:** never use `@`-imports as a token-reduction strategy. If the goal is reducing context, the correct moves are path-scoped rules or skills.

Mechanics to hold:
- Both relative and absolute paths work; relative paths resolve against the **file containing the import**, not the working directory `[memory]`.
- Recursive imports allowed, **maximum depth four hops** `[memory]`.
- Import parsing skips markdown code spans and fenced blocks. To mention a path without importing it, wrap it in backticks `[memory]`.
- An import in a **project-level** memory file whose path resolves outside the working directory is treated as external and triggers a one-time approval dialog. Imports in user-scope memory files load without the dialog `[memory]`.
- Block-level HTML comments in CLAUDE.md are stripped before injection — use them for human-maintainer notes at zero token cost `[memory]`.
- To share personal instructions across git worktrees, import from home (`@~/.claude/my-project-instructions.md`) rather than relying on a gitignored `CLAUDE.local.md`, which exists only in the worktree where it was created `[memory]`.

## 5. Output styles

**What they are for.** Changing **how Claude responds, not what Claude knows**. They modify the system prompt to set role, tone, and output format. Use one when you keep re-prompting for the same voice or format every turn, or when Claude should act as something other than a software engineer `[output-styles]`.

**Anthropic's explicit routing line:** *"For instructions about your project, conventions, or codebase, use CLAUDE.md instead."* `[output-styles]`

**When it loads.** Part of the system prompt, read **once at session start**. Changes take effect only after `/clear` or a new session `[output-styles]`. Note the asymmetry with `settings.json` generally: most settings keys reload live, but `outputStyle` and `model` are read once at session start `[settings]`.

**Precedence and locations** `[output-styles]`:
- User: `~/.claude/output-styles`
- Project: `.claude/output-styles`
- Managed policy: `.claude/output-styles` inside the managed settings directory
- Project styles load from every `.claude/output-styles/` between the working directory and repo root; when two nested directories define the same name, the one **closest to the working directory** wins (v2.1.178+)
- A plugin style with `force-for-plugin: true` applies automatically whenever the plugin is enabled and **overrides the user's `outputStyle` setting**

**The frontmatter decision that matters most:** `keep-coding-instructions`. Default is `false`, which means a custom output style **removes Claude Code's built-in software engineering instructions** — how to scope changes, write comments, and verify work. Set it to `true` when changing communication but still coding; leave it out when Claude isn't doing software engineering at all `[output-styles]`.

**Scope limit:** output styles apply to the **main conversation only**. A subagent runs its own system prompt, so styles don't shape subagent responses. A conversation fork is the exception, because it inherits the parent's full system prompt `[output-styles]`.

**Built-ins:** Default, Proactive, Explanatory, Learning. Proactive is stronger autonomous-execution guidance than auto mode applies, and works without changing permission mode `[output-styles]`.

## 6. Skills — and note that custom slash commands are now skills

**Custom commands have been merged into skills.** A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way. Existing `.claude/commands/` files keep working. Skills add a directory for supporting files, frontmatter to control who invokes them, and automatic loading when relevant. If a skill and a command share a name, **the skill takes precedence** `[skills]`.

**Directive:** write new invocable workflows as skills, not as `commands/` files. Treat "custom slash command" as a legacy name for a user-invocable skill.

**What skills are for.** Content Claude needs *sometimes*: reference material, or a repeatable procedure. Anthropic's trigger: create a skill when you keep pasting the same instructions into chat, **or when a section of CLAUDE.md has grown into a procedure rather than a fact** `[skills]`.

**When it loads — progressive disclosure, in three stages** `[skills]` `[skill-bp]`:
1. **Session start:** name + description only, into a listing so Claude knows what exists.
2. **On invocation:** the full rendered `SKILL.md` body enters the conversation as a single message.
3. **On demand:** supporting files in the skill directory are read individually, only when referenced. Bundled files consume **zero context tokens until actually read** `[skill-bp]`.

**Lifecycle detail with real consequences:** once invoked, the rendered SKILL.md content **stays in context for the rest of the session**, and Claude Code does not re-read the file on later turns. Write guidance that should apply throughout a task as **standing instructions**, not one-time steps `[skills]`.

**Precedence** `[skills]`:
- Enterprise (managed) **>** personal (`~/.claude/skills/`) **>** project (`.claude/skills/`)
- A skill at any of these levels overrides a **bundled** skill of the same name
- Plugin skills are namespaced `plugin-name:skill-name` and therefore cannot conflict
- Nested skills below the working directory load when Claude touches a file in that subtree, and appear under a directory-qualified name such as `apps/web:deploy` (v2.1.203+)

**The description field is the whole triggering mechanism.** Claude matches the task against skill descriptions to decide what to load. Constraints to respect `[skills]`:
- `description` + `when_to_use` are truncated at **1,536 characters** in the listing. Put the key use case first.
- The listing's total budget scales at **1% of the model's context window**. When it overflows, Claude Code drops descriptions starting with the **least-invoked** skills — silently stripping the keywords needed for matching.
- Diagnose with `/doctor` for an estimate of listing cost and the biggest contributors; raise with `skillListingBudgetFraction`; free budget by setting low-priority skills to `"name-only"` in `skillOverrides`.

**Invocation control** `[skills]`:

| Frontmatter | You invoke | Claude invokes | Context at startup |
|:---|:---|:---|:---|
| (default) | Yes | Yes | Description in context |
| `disable-model-invocation: true` | Yes | No | **Nothing** |
| `user-invocable: false` | No | Yes | Description in context |

**Directives:**
- Anything with side effects — commit, deploy, send message — gets `disable-model-invocation: true` `[skills]`.
- Background knowledge that isn't a meaningful user action gets `user-invocable: false` `[skills]`.
- Keep `SKILL.md` under **500 lines**; move detail to separate files `[skills]` `[skill-bp]`.
- Put the most important instructions **near the top** of SKILL.md, because compaction truncation keeps the start of the file `[context]`.

## 7. Subagents — separate context, separate instructions

**What they are for.** A side task that would otherwise flood the main conversation with search results, logs, or file contents you won't reference again. Define a custom subagent when you keep spawning the same kind of worker with the same instructions `[subagents]`.

**When it loads.** On spawn, into a **fresh, isolated context window**. The subagent does not see conversation history, previously invoked skills, or files already read `[subagents]`.

**What actually reaches a non-fork subagent** `[subagents]`:
- Its **own system prompt** (the markdown body of its definition file), plus environment details — **not** the full Claude Code system prompt
- The delegation task message Claude writes
- **Every level of the CLAUDE.md hierarchy**, including user, project rules, `CLAUDE.local.md`, and managed policy files
- Git status snapshot
- Full content of any skill named in the `skills:` field — **preloaded in full, not just the description**

**What never reaches it:** the main conversation's output style, the main conversation's auto memory, and the conversation history `[subagents]`.

**The two exceptions to memorize:** the built-in **Explore** and **Plan** agents skip CLAUDE.md and git status to keep their context small. There is **no frontmatter field or per-agent setting** to change which agents skip them `[subagents]`.

**Directive following from that:** the main conversation reads Explore and Plan results with full CLAUDE.md context, so most rules don't need to reach the subagent. If a rule *must* reach it — Anthropic's example is *"ignore the `vendor/` directory"* — **restate it in the delegation prompt** `[subagents]`.

**Precedence, highest first** `[subagents]`: managed settings → `--agents` CLI flag → `.claude/agents/` (project) → `~/.claude/agents/` (user) → plugin `agents/` directory.

Within a scope, identity comes only from the frontmatter `name` field, not the file path. Duplicate names inside one `.claude/agents/` tree resolve by **filesystem read order** — undocumented and unstable. Keep names unique; `/doctor` reports duplicates `[subagents]`.

**Plugin subagents silently ignore `hooks`, `mcpServers`, and `permissionMode`** for security reasons. If those are needed, copy the agent file into `.claude/agents/` or `~/.claude/agents/` `[subagents]`.

**Where instructions go inside a subagent:**
- `description` → decides *when Claude delegates*. Include "use proactively" to encourage delegation `[subagents]`.
- Markdown body → the system prompt; put the role, the checklist, and the output format here `[subagents]`.
- `skills:` → preload reference material rather than restating it in the body `[subagents]`.
- `tools` / `disallowedTools` → capability limits. `disallowedTools` is applied first, then `tools` resolves against what remains `[subagents]`.

**Forks are the exception to all of the above.** A fork inherits the entire conversation, system prompt, tools, and model, and shares the parent's prompt cache. Use a fork when a named subagent would need too much background to be useful `[subagents]`.

## 8. Hooks — the only deterministic layer for behavior

**What they are for.** Actions that must happen the same way every time and don't need Claude to think: format on save, reject a command, log, notify `[features]`.

**When it loads.** On the lifecycle event. **Zero context cost unless the hook returns output** `[features]`. Hook output reaches Claude only through specific channels: `hookSpecificOutput.additionalContext`, or stderr on exit code 2. Plain stdout on exit 0 goes to the debug log, not to Claude `[context]`.

**Precedence: hooks merge, they do not override.** All registered hooks fire for their matching events regardless of source `[features]`. When several match the same event, every command runs to completion before results are merged; for `PreToolUse` decisions the **most restrictive answer wins**, in the order `deny`, `defer`, `ask`, `allow` `[hooks-guide]`.

**Locations, by scope** `[hooks-guide]`:

| Location | Scope | Shareable |
|:---|:---|:---|
| `~/.claude/settings.json` | All your projects | No |
| `.claude/settings.json` | One project | Yes, committable |
| `.claude/settings.local.json` | One project | No |
| Managed policy settings | Organization | Admin-controlled |
| Plugin `hooks/hooks.json` | Where plugin enabled | Yes |
| Skill or agent frontmatter | While that component is active | Yes |

**The asymmetry that determines what a hook can enforce** `[hooks-guide]`:
- `PreToolUse` hooks fire **before any permission-mode check**, in every mode including `bypassPermissions` and `--dangerously-skip-permissions`. A hook returning `deny` blocks the tool regardless of mode.
- The reverse is **not** true. A hook returning `allow` does not bypass deny rules from settings.
- Stated plainly by Anthropic: *"Hooks can tighten restrictions but not loosen them past what permission rules allow."*

**Directive:** when a requirement is stated as "never" or "always, without exception," write it as a `PreToolUse` hook and, if it's also a capability boundary, back it with a `permissions.deny` rule. Do not write it only as CLAUDE.md prose.

**Two hook events worth knowing for instruction placement specifically:**
- `SessionStart` with matcher `compact` re-injects context after compaction — the correct home for instructions that must survive a long session but aren't in project-root CLAUDE.md `[hooks-guide]`.
- `InstructionsLoaded` fires when a CLAUDE.md or `.claude/rules/*.md` file loads, and logs which instruction files loaded, when, and why. This is the diagnostic tool for path-scoped and lazily loaded rules `[memory]`.

## 9. settings.json and permissions

**What belongs here.** Technical enforcement and configuration, not behavioral guidance `[memory]`.

**Settings precedence, highest first** `[settings]` `[permissions]`:
1. **Managed** — cannot be overridden by anything, including command-line arguments
2. **Command line arguments**
3. **Local** — `.claude/settings.local.json`
4. **Project** — `.claude/settings.json`
5. **User** — `~/.claude/settings.json`

**Permission rules behave differently from other settings: they merge across scopes rather than override** `[settings]`. Within the merged set:

- Rules are evaluated **deny → ask → allow**. First match in that order wins, and **rule specificity does not change the order** `[permissions]`.
- Consequence: a broad `Bash(aws *)` deny blocks a narrower `Bash(aws s3 ls)` allow. A deny rule cannot carry allowlist exceptions `[permissions]`.
- Deny at **any** level beats allow at any other level, in both directions: a user-level deny blocks a project-level allow, and a managed deny cannot be overridden by `--allowedTools` `[permissions]`.
- A **bare tool name** deny (`Bash`) removes the tool from Claude's context entirely; a **scoped** deny (`Bash(rm *)`) leaves the tool visible and blocks matching calls `[permissions]`.

**When edits take effect:** most keys, including `permissions` and `hooks`, reload live in the running session. `model` and `outputStyle` are read once at session start `[settings]`.

**Workspace trust gate:** `permissions.allow` rules and `additionalDirectories` in a project's `.claude/settings.json` grant capability, so they apply only after the workspace trust dialog is accepted. `deny` and `ask` rules are unaffected, since they only restrict `[permissions]`.

**Managed-only keys relevant to instruction authoring** `[permissions]` `[settings]`:
- `claudeMd` — inject org-wide CLAUDE.md content
- `allowManagedPermissionRulesOnly` — user and project settings cannot define allow/ask/deny
- `allowManagedHooksOnly` — only managed, SDK, and force-enabled plugin hooks load
- `strictPluginOnlyCustomization` — blocks skills, agents, hooks, and MCP servers from user and project sources; accepts `true` or an array such as `["skills", "hooks"]`
- `disableSkillShellExecution` — replaces `` !`cmd` `` blocks in user/project/plugin skills with a policy notice; bundled and managed skills unaffected

**Anthropic's warning about trying to enforce policy through Bash patterns:** patterns that constrain command arguments are fragile. `Bash(curl http://github.com/ *)` misses flag reordering, protocol changes, redirects, variables, and extra spaces. Use deny rules on the network tools plus `WebFetch(domain:...)` allows, or a `PreToolUse` hook. CLAUDE.md guidance *"shapes what Claude tries but doesn't enforce a boundary, so pair it with one of the options above"* `[permissions]`.

## 10. Plugins and MCP configuration

### Plugins — the packaging layer, not a new instruction type

A plugin bundles skills, hooks, subagents, MCP servers, LSP servers, monitors, and output styles into one installable unit `[features]` `[plugins]`.

**Anthropic's routing table** `[plugins]`:

| Approach | Skill names | Best for |
|:---|:---|:---|
| Standalone (`.claude/`) | `/hello` | Personal workflows, project-specific customization, quick experiments |
| Plugin | `/plugin-name:hello` | Sharing with teammates, distributing to community, versioned releases, reuse across projects |

**Directive:** start standalone in `.claude/` for iteration, convert to a plugin when ready to share `[plugins]`. Placement inside a plugin is otherwise identical to placement outside it — a plugin does not change *what kind* of instruction belongs where.

**Plugin-specific precedence facts:**
- Plugin subagents sit at the **lowest** priority (5 of 5) `[subagents]`
- Plugin skills are namespaced and never conflict `[skills]`
- `skillOverrides` does **not** affect plugin skills; manage those through `/plugin` `[skills]`
- Project and user `.claude/agents/` definitions **override** same-named plugin agents, so after migrating, remove the originals or the plugin version never takes effect `[plugins]`
- A plugin `settings.json` supports only `agent` and `subagentStatusLine`; `agent` activates one of the plugin's subagents as the main thread `[plugins]`
- Structural gotcha: only `plugin.json` goes inside `.claude-plugin/`. `skills/`, `agents/`, `hooks/`, `.mcp.json` all live at the plugin **root** `[plugins]`

### MCP server configuration

**Scopes and precedence, highest first** `[mcp]`: local → project → user → plugin-provided → claude.ai connectors.

The entire server entry from the highest-precedence source is used; **fields are not merged across scopes**. The three scopes match duplicates by name; plugins and connectors match by **endpoint** `[mcp]`.

| Scope | Loads in | Shared | Stored in |
|:---|:---|:---|:---|
| Local (default) | Current project only | No | `~/.claude.json` |
| Project | Current project only | Yes, via VCS | `.mcp.json` at project root |
| User | All your projects | No | `~/.claude.json` |

**Placement-relevant context facts:**
- Tool search is **on by default**: only tool names and server instructions load at start, and full schemas defer until needed. Idle MCP tools therefore consume minimal context `[mcp]` `[features]`.
- To keep a server's tool descriptions out of the main conversation entirely, **define it inline in a subagent's `mcpServers` frontmatter** rather than in `.mcp.json`. The subagent gets the tools; the parent does not `[subagents]`.
- For MCP server authors: server instructions become more useful with tool search on, and function like a skill description — explain what category of tasks the tools handle and when Claude should search for them. **Tool descriptions and server instructions truncate at 2KB each; put critical details near the start** `[mcp]`.
- `alwaysLoad: true` exempts a server from deferral, loading every tool upfront and blocking startup until connect (5s cap). Use sparingly `[mcp]`.

**Where instructions about an MCP server go:** in a **skill**, not in the server config. Anthropic's pattern: MCP provides the connection; a skill teaches Claude the schema, query patterns, and which tables to use `[features]`.

**MCP tool naming for permission rules, hook matchers, and `allowed-tools`:** plugin-bundled servers use `mcp__plugin_<plugin-name>_<server-name>__<tool-name>`. A matcher written against the bare server key never fires for a plugin-bundled server `[mcp]`. In skill bodies, use fully qualified `ServerName:tool_name` references `[skill-bp]`.

## 11. Session-level steering

These are not persisted instructions. They are the correct home for anything true only right now.

**Initial prompt.** Anthropic's guidance is that specificity in the prompt reduces correction cycles; see Part Two, Section 13 for the verbatim before/after pairs `[cc-best]`.

**Plan mode.** Separates exploration from execution so Claude doesn't solve the wrong problem. Claude reads files and runs read-only commands but doesn't edit source `[cc-best]` `[permissions]`. Anthropic is explicit that it has a cost: *"For tasks where the scope is clear and the fix is small (like fixing a typo, adding a log line, or renaming a variable) ask Claude to do it directly... If you could describe the diff in one sentence, skip the plan."* `[cc-best]`

Plan mode delegates research to the **Plan subagent**, which skips CLAUDE.md — so project conventions reach the main conversation reading the results, not the researcher `[subagents]`.

**`/compact`.** Replaces the conversation with a structured summary. Prefer `/compact <instructions>` — e.g. `/compact focus on the auth bug fix` — before a long new task, so the summary keeps what you choose rather than what the automatic pass guesses `[context]`.

**`/clear`.** Reset between unrelated tasks. Anthropic's specific threshold: *"If you've corrected Claude more than twice on the same issue in one session, the context is cluttered with failed approaches. Run `/clear` and start fresh with a more specific prompt that incorporates what you learned."* `[cc-best]`

**What survives compaction** — consult this table before deciding an instruction is safe in a lazily loaded location `[context]`:

| Mechanism | After compaction |
|:---|:---|
| System prompt and output style | Unchanged; not part of message history |
| Project-root CLAUDE.md and unscoped rules | Re-injected from disk |
| Auto memory | Re-injected from disk |
| Rules with `paths:` frontmatter | **Lost** until a matching file is read again |
| Nested CLAUDE.md in subdirectories | **Lost** until a file in that subdirectory is read again |
| Invoked skill bodies | Re-injected, capped 5,000 tokens per skill and 25,000 total; oldest dropped first |
| Hooks | Not applicable; hooks run as code, not context |
| Skill **descriptions** listing | **Not re-injected**; only skills actually invoked are preserved |

**Directives from that table:**
- If a rule must persist across compaction, drop the `paths:` frontmatter or move it to project-root CLAUDE.md `[context]`.
- Put the highest-value instructions at the top of a large SKILL.md, since truncation keeps the start of the file `[context]`.
- After a compaction in a long session, re-invoke a large skill to restore its full content `[skills]`.
- CLAUDE.md can carry compaction instructions, e.g. *"When compacting, always preserve the full list of modified files and any test commands"* `[cc-best]`.

## 12. Precedence at a glance

| Construct | Resolution model | Order (highest first) |
|:---|:---|:---|
| CLAUDE.md / rules | **Additive**; conflicts by model judgment | Concatenated root→cwd; more specific typically wins `[features]` |
| Skills | Override by name | Enterprise → personal → project → bundled; plugin namespaced `[skills]` |
| Subagents | Override by name | Managed → `--agents` → project → user → plugin `[subagents]` |
| Output styles | Override by name | Managed / project / user dirs; nested closest-to-cwd wins `[output-styles]` |
| MCP servers | Override by name (whole entry) | Local → project → user → plugin → claude.ai connector `[mcp]` |
| Hooks | **Merge**; all fire | Most restrictive `PreToolUse` decision wins `[features]` `[hooks-guide]` |
| Settings | Override by key | Managed → CLI args → local → project → user `[settings]` |
| Permission rules | **Merge**, then evaluate | deny → ask → allow; first match wins; specificity irrelevant `[permissions]` |

**Note the split:** four constructs override, two merge, and CLAUDE.md does neither — it concatenates and leaves conflicts to judgment. Do not carry an intuition from one row to another.

---

# PART TWO — PHRASING

Placement decides whether Claude sees an instruction. Phrasing decides whether Claude follows it. These are separate failures with separate fixes: if Claude never mentions a rule, suspect placement; if Claude sees it and behaves inconsistently, suspect phrasing.

## 13. Specificity — the highest-yield lever

**Anthropic's golden rule:** *"Show your prompt to a colleague with minimal context on the task and ask them to follow it. If they'd be confused, Claude will be too."* `[prompt-bp]`

**The framing to reason from:** *"Think of Claude as a brilliant but new employee who lacks context on your norms and workflows."* `[prompt-bp]`

### Verbatim before/after pairs for standing instructions `[cc-best]`

| Before | After |
|:---|:---|
| "Format code properly" | "Use 2-space indentation" |
| "Test your changes" | "Run `npm test` before committing" |
| "Keep files organized" | "API handlers live in `src/api/handlers/`" |

Anthropic's stated criterion is verifiability: *"write instructions that are concrete enough to verify"* `[memory]`.

### Verbatim before/after pairs for prompts `[cc-best]`

| Strategy | Before | After |
|:---|:---|:---|
| Scope the task | *"add tests for foo.py"* | *"write a test for foo.py covering the edge case where the user is logged out. avoid mocks."* |
| Point to sources | *"why does ExecutionFactory have such a weird api?"* | *"look through ExecutionFactory's git history and summarize how its api came to be"* |
| Reference existing patterns | *"add a calendar widget"* | *"look at how existing widgets are implemented on the home page to understand the patterns. HotDogWidget.php is a good example. follow the pattern to implement a new calendar widget that lets the user select a month and paginate forwards/backwards to pick a year. build from scratch without libraries other than the ones already used in the codebase."* |
| Describe the symptom | *"fix the login bug"* | *"users report that login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces the issue, then fix it"* |
| Provide verification criteria | *"implement a function that validates email addresses"* | *"write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing"* |
| Verify UI visually | *"make the dashboard look better"* | *"[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them"* |
| Address root causes | *"the build is failing"* | *"the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error"* |

**One caveat Anthropic states rather than hiding:** *"Vague prompts can be useful when you're exploring and can afford to course-correct. A prompt like `'what would you improve in this file?'` can surface things you wouldn't have thought to ask about."* `[cc-best]`

## 14. Positive versus negative framing

**Anthropic's stated technique, verbatim** `[prompt-bp]`:

> **Tell Claude what to do instead of what not to do**
> - Instead of: "Do not use markdown in your response"
> - Try: "Your response should be composed of smoothly flowing prose paragraphs."

Two supporting techniques from the same section `[prompt-bp]`:
- **Use XML format indicators:** *"Write the prose sections of your response in `<smoothly_flowing_prose_paragraphs>` tags."*
- **Match prompt style to desired output:** *"removing markdown from your prompt can reduce the volume of markdown in the output."*

**Directive:** convert every prohibition into the positive behavior that replaces it. Where a prohibition genuinely has no positive form — a hard boundary such as "never edit `.env`" — that is a signal the instruction belongs in a hook or a deny rule, not in prose (see Part One, Section 1).

**Honest limit:** Anthropic states the positive-framing rule for *output formatting* specifically. It does not publish a general claim that negatively framed instructions are followed less reliably in all contexts, and Anthropic's own sample prompts do use negative framing where the constraint is a genuine boundary — e.g. *"Don't add features, refactor code, or make 'improvements' beyond what was asked"* and *"Do not hard-code values"* `[prompt-bp]`. Treat positive framing as the default and negative framing as acceptable for true boundaries.

## 15. Emphasis and its failure modes — sources conflict, resolve deliberately

**This is the one point where Anthropic's own guidance points in two directions. Do not present either side as settled.**

**Position A — emphasis helps adherence** `[cc-best]`, Claude Code best practices:

> "You can tune instructions by adding emphasis (e.g., 'IMPORTANT' or 'YOU MUST') to improve adherence."

The skill-authoring guide describes the same escalation as a legitimate iteration step: when a rule isn't being followed, Claude *"might suggest reorganizing to make rules more prominent, using stronger language such as 'MUST filter' instead of 'always filter,' or restructuring the workflow section"* `[skill-bp]`.

**Position B — emphasis causes overtriggering** `[prompt-bp]`, the prompting best-practices reference:

> "Claude Opus 4.5 and Claude Opus 4.6 are also more responsive to the system prompt than previous models. If your prompts were designed to reduce undertriggering on tools or skills, these models may now overtrigger. The fix is to dial back any aggressive language. Where you might have said 'CRITICAL: You MUST use this tool when...', you can use more normal prompting like 'Use this tool when...'."

The same page adds a second overtriggering pattern `[prompt-bp]`:
- *"Replace blanket defaults with more targeted instructions. Instead of 'Default to using [tool],' add guidance like 'Use [tool] when it would enhance your understanding of the problem.'"*
- *"Remove over-prompting. Tools that undertriggered in previous models are likely to trigger appropriately now. Instructions like 'If in doubt, use [tool]' will cause overtriggering."*

**Which is more current and more authoritative.** `[prompt-bp]` is model-versioned, names current models explicitly, and was the more recently revised of the two at the time of compilation. `[cc-best]` is surface-specific to Claude Code and does not carry model qualifiers. Neither supersedes the other outright: they address different failure modes. `[cc-best]` addresses a rule being *ignored*; `[prompt-bp]` addresses a tool or skill firing *too often*.

**Operating rule to follow until this is resolved by evidence:**
1. Do not reach for emphasis first. Anthropic's stated first diagnosis when a rule is ignored is that **the file is too long and the rule is getting lost**: *"If Claude keeps doing something you don't want despite having a rule against it, the file is probably too long and the rule is getting lost."* `[cc-best]`
2. Second diagnosis: the phrasing is ambiguous. *"If Claude asks you questions that are answered in CLAUDE.md, the phrasing might be ambiguous."* `[cc-best]`
3. Only after pruning and disambiguating, add emphasis — and add it to **one** rule, not several. Emphasis distributed across many rules is indistinguishable from no emphasis.
4. If the rule must hold every time, stop tuning prose and write a hook. Emphasis is a probability adjustment; a hook is a guarantee `[features]`.

**What would resolve the disagreement:** a with/without A-B comparison on the specific rule, run in fresh sessions. Anthropic ships the tooling for exactly this — the `skill-creator` plugin runs blind A/B between two skill versions and reports pass-rate delta against token and time overhead `[skills]`. Absent that measurement, treat any claim that emphasis helped as unverified.

## 16. Give rationale

**Anthropic's stated technique:** *"Providing context or motivation behind your instructions, such as explaining to Claude why such behavior is important, can help Claude better understand your goals and deliver more targeted responses."* Followed by: *"Claude is smart enough to generalize from the explanation."* `[prompt-bp]`

**The counterweight, from the same body of guidance:** Anthropic's CLAUDE.md exclusion list explicitly excludes *"Long explanations or tutorials"* `[cc-best]`, and the skill guide's core principle is *"Only add context Claude doesn't already have"* with the challenge questions *"Does Claude really need this explanation?"* / *"Does this paragraph justify its token cost?"* `[skill-bp]`.

**Reconciliation directive:** give rationale where the rule would otherwise look arbitrary or over-broad, and keep it to a clause. `[cc-best]`'s own include-list resolves this: keep *"Common gotchas or non-obvious behaviors"* and *"Architectural decisions specific to your project"* — rationale that is itself project knowledge. Cut rationale that restates general software engineering.

Worked example of the boundary:
- **Rationale that earns its tokens:** "Run `make lint` before committing — CI rejects unformatted commits and the retry costs 10 minutes."
- **Rationale that does not:** "Run `make lint` before committing. Linting is important because consistent formatting helps teams read code more easily and reduces merge conflicts."

## 17. Examples

**Anthropic calls examples the most reliable steering mechanism:** *"Examples are one of the most reliable ways to steer Claude's output format, tone, and structure."* `[prompt-bp]`

Requirements `[prompt-bp]`:
- **Relevant** — mirror the actual use case closely
- **Diverse** — cover edge cases and vary enough that Claude doesn't pick up unintended patterns
- **Structured** — wrap in `<example>` tags (multiple in `<examples>`) so Claude can distinguish them from instructions
- **Count:** include **3–5** for best results

For skills, Anthropic's guidance is input/output pairs: *"For Skills where output quality depends on seeing examples, provide input/output pairs just like in regular prompting."* Their worked example gives three commit-message pairs, then names the pattern: *"Follow this style: type(scope): brief description, then detailed explanation."* And the justification: *"Examples convey the desired style and level of detail to Claude more clearly than descriptions alone."* `[skill-bp]`

**Directive:** when an instruction has been rewritten twice and behavior still varies, replace the description with three examples rather than rewriting a third time.

## 18. Structure and headers

**In CLAUDE.md:** *"use markdown headers and bullets to group related instructions. Claude scans structure the same way readers do: organized sections are easier to follow than dense paragraphs."* `[memory]`

**In skills, structure is load-bearing, not cosmetic** `[skill-bp]`:
- **Keep references one level deep from SKILL.md.** Claude may partially read files reached through nested references, using commands like `head -100` to preview rather than reading whole files, producing incomplete information.
  - Bad: `SKILL.md` → `advanced.md` → `details.md` → the actual information
  - Good: `SKILL.md` links directly to `advanced.md`, `reference.md`, `examples.md`
- **Add a table of contents to any reference file longer than 100 lines**, so Claude sees the full scope even when previewing with a partial read.
- **Name files descriptively:** `form_validation_rules.md`, not `doc2.md`. Organize by domain: `reference/finance.md`, `reference/sales.md`, not `docs/file1.md`.
- **Use forward slashes in every path**, even on Windows.
- **Make execution intent explicit:** *"Run `analyze_form.py` to extract fields"* (execute) versus *"See `analyze_form.py` for the extraction algorithm"* (read as reference).

**For structured prompts generally:** XML tags help Claude parse prompts that mix instructions, context, examples, and inputs. Use consistent, descriptive tag names; nest when content has natural hierarchy `[prompt-bp]`.

**Long-context ordering:** put longform data **at the top**, above the query and instructions. Anthropic reports queries at the end improving response quality by up to 30% in tests on complex multidocument inputs `[prompt-bp]`.

## 19. Length

**Hard targets Anthropic publishes:**

| Artifact | Target | Source |
|:---|:---|:---|
| CLAUDE.md | Under **200 lines** | `[features]` `[memory]` |
| SKILL.md body | Under **500 lines** | `[skills]` `[skill-bp]` |
| Skill `description` + `when_to_use` | Under **1,536 characters** (hard truncation) | `[skills]` |
| Auto memory `MEMORY.md` | First **200 lines or 25KB** loads | `[memory]` |
| MCP tool descriptions / server instructions | **2KB** each (hard truncation) | `[mcp]` |

**The test to apply line by line:** *"For each line, ask: 'Would removing this cause Claude to make mistakes?' If not, cut it. Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"* `[cc-best]`

**Anthropic's include/exclude table for CLAUDE.md, reproduced** `[cc-best]`:

| ✅ Include | ❌ Exclude |
|:---|:---|
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Code style rules that differ from defaults | Standard language conventions Claude already knows |
| Testing instructions and preferred test runners | Detailed API documentation (link to docs instead) |
| Repository etiquette (branch naming, PR conventions) | Information that changes frequently |
| Architectural decisions specific to your project | Long explanations or tutorials |
| Developer environment quirks (required env vars) | File-by-file descriptions of the codebase |
| Common gotchas or non-obvious behaviors | Self-evident practices like "write clean code" |

**Anthropic's conciseness principle for skills, with a verbatim token comparison** `[skill-bp]`:

> **Good example: Concise** (approximately 50 tokens):
> ```
> ## Extract PDF text
>
> Use pdfplumber for text extraction:
> [code block]
> ```
>
> **Bad example: Too verbose** (approximately 150 tokens):
> ```
> ## Extract PDF text
>
> PDF (Portable Document Format) files are a common file format that contains
> text, images, and other content. To extract text from a PDF, you'll need to
> use a library. There are many libraries available for PDF processing, but
> pdfplumber is recommended because it's easy to use and handles most cases well.
> First, you'll need to install it using pip. Then you can use the code below...
> ```

Their stated reason: *"The concise version assumes Claude already has information about PDFs and how libraries work."* The governing principle: *"The context window is a public good."* `[skill-bp]`

**Automated help:** `/doctor` proposes trims for a checked-in CLAUDE.md — it cuts content Claude can derive from the codebase, such as directory layouts, dependency lists, and architecture overviews, and keeps pitfalls, rationale, and conventions that differ from tool defaults. Requires v2.1.206+ `[memory]`.

## 20. Writing descriptions that trigger correctly

The `description` field is the entire discovery mechanism for skills and subagents. Treat it as a separate authoring task from the body.

**Always write in third person** `[skill-bp]`. The description is injected into the system prompt, and inconsistent point-of-view causes discovery problems.
- **Good:** "Processes Excel files and generates reports"
- **Avoid:** "I can help you process Excel files"
- **Avoid:** "You can use this to process Excel files"

**State both what it does and when to use it.** Verbatim effective examples `[skill-bp]`:

```
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```
```
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```
```
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

**Verbatim vague descriptions to avoid** `[skill-bp]`:
```
description: Helps with documents
description: Processes data
description: Does stuff with files
```

**Naming:** prefer gerund form — `processing-pdfs`, `analyzing-spreadsheets`, `testing-code`. Acceptable alternatives are noun phrases (`pdf-processing`) or action-oriented (`process-pdfs`). Avoid `helper`, `utils`, `tools`, `documents`, `data`, `files`. `name` is max 64 characters, lowercase letters/numbers/hyphens only, and cannot contain the reserved words "anthropic" or "claude" `[skill-bp]`.

**Troubleshooting triggering** `[skills]`:
- *Not triggering:* check the description includes keywords users would naturally say; verify the skill appears when you ask "What skills are available?"; check whether description truncation stripped the keywords.
- *Triggering too often:* make the description more specific, or add `disable-model-invocation: true`.

**For subagents, the description decides delegation.** Include "use proactively" to encourage it `[subagents]`. Anthropic's worked example: *"Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code."* `[subagents]`

## 21. Match specificity to the task's fragility

Anthropic frames this as **degrees of freedom** `[skill-bp]`. The analogy: *"Narrow bridge with cliffs on both sides: there's only one safe way forward. Provide specific guardrails and exact instructions (low freedom)... Open field with no hazards: many paths lead to success. Give general direction and trust Claude to find the best route (high freedom)."*

| Freedom | Use when | Form |
|:---|:---|:---|
| **High** | Multiple approaches valid; decisions depend on context | Text instructions: *"1. Analyze the code structure and organization / 2. Check for potential bugs or edge cases..."* |
| **Medium** | A preferred pattern exists; some variation acceptable | Pseudocode or a parameterized script |
| **Low** | Operations fragile; consistency critical; exact sequence required | *"Run exactly this script: `python scripts/migrate.py --verify --backup`. Do not modify the command or add additional flags."* |

Same idea applied to templates `[skill-bp]`: for strict requirements, *"ALWAYS use this exact template structure"*; for flexible guidance, *"Here is a sensible default format, but use your best judgment based on the analysis."*

**Directive:** before writing a procedure, decide which of the three it is. Writing a low-freedom instruction for an open-field task produces brittle behavior; writing a high-freedom instruction for a fragile task produces failures.

## 22. Consistency, terminology, and anti-patterns

**Use one term throughout** `[skill-bp]`:
- Good: always "API endpoint", always "field", always "extract"
- Bad: mixing "API endpoint"/"URL"/"API route"/"path"; mixing "field"/"box"/"element"/"control"; mixing "extract"/"pull"/"get"/"retrieve"

**Avoid time-sensitive information.** Verbatim bad example `[skill-bp]`:
```
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```
Verbatim good pattern: a `## Current method` section, with legacy content in a collapsed `## Old patterns` section.

**Avoid offering too many options.** Verbatim `[skill-bp]`:
> **Bad example: Too many choices** (confusing): "You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."
> **Good example: Provide a default** (with escape hatch): "Use pdfplumber for text extraction... For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."

**Don't assume tools are installed.** Bad: *"Use the pdf library to process the file."* Good: *"Install required package: `pip install pypdf`"* then the usage `[skill-bp]`.

**Anthropic's named failure patterns for instruction files** `[cc-best]`:
- *The over-specified CLAUDE.md.* "If your CLAUDE.md is too long, Claude ignores half of it because important rules get lost in the noise." Fix: *"Ruthlessly prune. If Claude already does something correctly without the instruction, delete it or convert it to a hook."*
- *The kitchen sink session.* Fix: `/clear` between unrelated tasks.
- *Correcting over and over.* Fix: after two failed corrections, `/clear` and write a better initial prompt.

**Treat instruction files as code:** *"review it when things go wrong, prune it regularly, and test changes by observing whether Claude's behavior actually shifts."* `[cc-best]`

## 23. Verify a phrasing change actually worked

Anthropic is explicit that observing a trigger is not evidence of correctness: *"Seeing a skill trigger tells you Claude found it, not that it did what you intended. To know a skill is working, measure two things separately: whether Claude invokes it on the prompts it should, and whether the output matches what you expect when it does."* `[skills]`

**The method:** collect a few realistic prompts, run each in a **fresh session** with the instruction present and again with it disabled, and compare. *"A fresh session matters because leftover context from authoring the skill will mask gaps in the written instructions."* `[skills]`

**Build evaluations before writing extensive documentation** `[skill-bp]`: run Claude on representative tasks without the instruction and document specific failures; build three scenarios testing those gaps; establish a baseline; write the minimum content that passes; iterate. *"This approach ensures you're solving actual problems rather than anticipating requirements that may never materialize."*

**Tooling:** the `skill-creator` plugin automates the loop inside Claude Code — test cases in `evals/evals.json`, one subagent per case for clean context, pass/fail grading with evidence, with-skill versus without-skill benchmarking, blind A/B between two versions, and description tuning that generates should-trigger and should-not-trigger prompts and proposes edits when the skill activates on the wrong requests. Install with `/plugin install skill-creator@claude-plugins-official`, then `/reload-plugins` `[skills]`.

**Diagnostics for the placement half:**
- `/context` — what actually loaded, by category `[context]`
- `/memory` — which CLAUDE.md and auto-memory files loaded `[memory]`
- `/doctor` — skill listing cost, biggest contributors, CLAUDE.md trim proposals, duplicate agent names `[skills]` `[memory]` `[subagents]`
- `--debug` — YAML parse errors, skill listing overflow warnings, hook execution detail `[skills]` `[hooks-guide]`
- `InstructionsLoaded` hook — logs exactly which instruction files loaded, when, and why `[memory]`

---

# PART THREE — MAPPING TO CLAUDE.AI

## 24. Which Claude Code construct maps to which claude.ai feature

claude.ai has three personalization layers: **profile preferences** (account-wide), **project instructions** (per project), and **styles** (how Claude formats and delivers responses) `[sup-personalization]`.

| Claude Code | claude.ai equivalent | Fidelity of the mapping |
|:---|:---|:---|
| `~/.claude/CLAUDE.md` | **Profile preferences** (Settings → "What preferences should Claude consider in responses?") — applied to all conversations `[sup-personalization]` | Close. Both are always-on, user-scoped, advisory. |
| Project `./CLAUDE.md` | **Project instructions** ("Set project instructions") — used for all chats within the project `[sup-projects]` `[sup-personalization]` | Close. Both are always-on and project-scoped. |
| `@`-imported reference docs; unscoped `.claude/rules/` | **Project knowledge** (uploaded files) `[sup-projects]` | Partial, and the loading model differs — see below. |
| Output styles | **Styles** `[sup-personalization]` | Close in intent. Both target *how* Claude responds rather than what it knows. |
| Skills | **Skills** enabled for the claude.ai account `[skills]` | Close. Cowork and cloud sessions load account skills, synced at session start. |
| MCP servers | **Connectors** (claude.ai/customize/connectors) `[mcp]` | Close; connectors surface in Claude Code automatically under a claude.ai login. |
| Managed policy CLAUDE.md | Shared projects on Team/Enterprise `[sup-projects]` | Weak. Sharing distributes instructions; it does not enforce policy. |

**The one real divergence in the close mappings:** project knowledge is not simply "imports." On paid plans, when project knowledge approaches the context window limit, Claude **automatically enables RAG mode** to expand capacity `[sup-projects]`. Claude Code has no analogue — CLAUDE.md and its imports load in full at launch, and Anthropic states that imports do not reduce context `[memory]`. Content that is safely large in project knowledge is not safely large in CLAUDE.md.

## 25. Where no claude.ai equivalent exists

These have **no equivalent** in the claude.ai chat interface. If an instruction depends on one of them, it cannot be ported:

| Claude Code construct | Why there is no equivalent |
|:---|:---|
| **Hooks** | claude.ai exposes no lifecycle event system. Nothing in the chat interface is deterministic in the way a `PreToolUse` hook is. |
| **`settings.json` permissions** | No allow/ask/deny rule system. Connector tool controls exist at the organization level, but there is no per-user rule syntax. |
| **Subagents** | No user-definable isolated worker with its own system prompt, tools, and model. |
| **Plan mode** | No read-only exploration mode that gates edits. |
| **`/compact` and `/clear`** | Context is managed per chat; the equivalent action is starting a new chat, which is not the same as a directed summary. |
| **`CLAUDE.local.md`** | No personal-and-private layer within a shared project. Project instructions are shared with everyone who can see the project. |
| **`paths:`-scoped rules** | No mechanism to load an instruction only when a particular kind of file is touched. |
| **CLAUDE.md precedence chain** | Profile and project instructions stack in two layers; there is no directory hierarchy, no managed-policy tier at the instruction level, and no `claudeMdExcludes`. |

**The load-bearing consequence, and the thing to carry between surfaces:**

> On claude.ai, **every** customization is advisory context. There is no enforcement layer. In Claude Code, hooks and permission rules are enforcement, and everything else is advisory.

An instruction that only works in Claude Code because a hook backs it will silently become a suggestion when moved to a project's instructions. When porting a rule in either direction, first classify it as advisory or enforced, and state explicitly that the enforcement does not travel.

**Two smaller asymmetries worth holding:**
- Cowork and cloud sessions **do not read `~/.claude/skills/`** on your machine. A personal skill must be enabled for the claude.ai account, committed to the repo's `.claude/skills/`, or shipped in a repo-declared plugin `[skills]`.
- Free accounts can create a maximum of five projects `[sup-projects]`. The support article on personalization is less current on this point — an older revision described projects as paid-only `[sup-personalization]`. The projects article is the more specific and more recently updated of the two; prefer it.

## 26. Version-specific and surface-specific guidance

**Claims below are version-pinned. Re-check any of them against a running `claude --version` before relying on them.**

| Behavior | Applies from |
|:---|:---|
| `/output-style` command removed; use `/config` or the `outputStyle` setting | Deprecated v2.1.73, removed v2.1.91 `[output-styles]` |
| Nested output style: closest to working directory wins | v2.1.178 `[output-styles]` |
| Re-invoking an identical skill appends a note, not a second copy | v2.1.202 `[skills]` |
| Directory-qualified nested skill names (`apps/web:deploy`) | v2.1.203 `[skills]` |
| `/doctor` proposes CLAUDE.md trims | v2.1.206 `[memory]` |
| `/verify` and `/code-review` are user-invoked only | v2.1.215 `[skills]` |
| Subagents nest three layers deep by default | v2.1.219; was five in v2.1.172–2.1.216, one in v2.1.217–2.1.218 `[subagents]` |
| `/agents` no longer opens a creation wizard | v2.1.198 `[subagents]` |
| Project-level subagent frontmatter hooks require workspace trust | v2.1.218 `[subagents]` |

**Surface differences to state explicitly whenever they matter:**

- **Claude Code vs. the Agent SDK.** `[UNVERIFIED]` — the Claude Code documentation index lists an Agent SDK page covering "Control filesystem settings with `settingSources`" and a migration-guide entry titled "Settings sources default," which indicates the SDK's loading of filesystem settings differs from the CLI's. **The compiling session did not fetch those pages, so nothing here should be relied on for SDK behavior.** Treat it only as a flag: do not assume a CLAUDE.md-based instruction reaches an SDK agent without checking https://code.claude.com/docs/en/agent-sdk/claude-code-features.md first.
- **Model-specific phrasing guidance is real and is published per model.** Anthropic maintains separate pages for prompting Claude Opus 5, Sonnet 5, Opus 4.8, and Fable 5 `[prompt-bp]`. Two examples with direct authoring consequences: Claude Opus 5's default responses run longer than prior models' and effort does not reliably change visible length, so prompt explicitly for concision; and Opus 5 verifies its own work well without instruction, so verification instructions carried over from earlier-model prompts cause over-verification and should be **removed rather than rewritten** `[prompt-bp]`.
- **Emphasis guidance is the clearest case of surface-vs-model divergence.** See Section 15.

---

# PART FOUR — SELF-CHECK FOR SESSIONS USING THIS FILE

## 27. Before writing or revising any instruction file

Run this checklist. It is the operational form of Parts One and Two.

1. **Classify first.** Advisory or enforced? If the requirement is "always" or "never" without exception, it is a hook or a deny rule, not prose `[features]`.
2. **Check the load model.** Will this be needed every session, or sometimes? Every-session and short → CLAUDE.md. Sometimes → skill. Every session but topic-scoped → `.claude/rules/`. File-type-scoped → `paths:` frontmatter `[features]` `[memory]`.
3. **Apply the deletion test** to every line: would removing it cause Claude to make mistakes? `[cc-best]`
4. **Check the length target** for the artifact type (Section 19).
5. **Convert prohibitions to positive statements** unless the prohibition is a true boundary (Section 14).
6. **Make every instruction verifiable** — concrete enough that you could check compliance (Section 13).
7. **Scan for contradictions** against the other files in the chain. Conflicts resolve by model judgment, so a contradiction is a coin flip, not an override `[memory]`.
8. **Write the description separately from the body** for any skill or subagent, in third person, stating what it does and when to use it (Section 20).
9. **Do not add emphasis before pruning and disambiguating** (Section 15).
10. **Name what would falsify the change** — the fresh-session A/B that would show it worked (Section 23).

## 28. Two queries that should produce different behavior if this file is loaded

Run each in a fresh session in this project. If the file is working, the response should differ from a session without it in the specific way named.

**Test 1 — enforcement routing.**
> "I want Claude to never commit without running tests first. Where should I put that?"

*Expected with this file loaded:* routes to a **`PreToolUse` or `Stop` hook**, not to CLAUDE.md, and states the reason — an instruction in CLAUDE.md is a request, not a guarantee. Should not simply suggest adding "always run tests before committing" to CLAUDE.md.

**Test 2 — length and load-model pushback.**
> "Add our full REST API style guide to CLAUDE.md so Claude always follows it."

*Expected with this file loaded:* pushes back on placement and proposes a **skill** (or a `paths:`-scoped rule), citing the 200-line target, the every-request cost of CLAUDE.md, and the finding that bloated CLAUDE.md files cause Claude to ignore actual instructions. Should also note that `@`-importing the style guide would **not** solve the cost problem.

A third, weaker signal: asking "should I put IMPORTANT in front of this rule?" should produce the two-sided answer from Section 15 rather than a confident yes or no.
