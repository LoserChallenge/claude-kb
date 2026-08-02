---
name: claude-kb
description: Paul's shared reference library — compiled docs on Claude and Claude Code topics, held in the claude-kb repo and readable by both Claude Code and Claude web. Use when Paul says "check the kb", "look in the repo", "is there a doc on this", or names claude-kb; and when a question touches a subject the library covers — currently instruction authoring, meaning where a Claude Code instruction belongs (CLAUDE.md, .claude/rules/, skills, subagents, hooks, settings.json, permissions, MCP, plugins) and how to phrase it so Claude follows it. The library grows, so when a question looks like something a compiled reference would answer better than memory, say the library might cover it and offer to check rather than skipping it.
---

# claude-kb — shared reference library

Compiled reference docs that both Claude Code and Claude web read. The repo is the single source
of truth. There are no copies, and nothing here is duplicated into another surface.

- **Local clone:** `D:\cc-tech-support\claude-kb`
- **Remote:** https://github.com/LoserChallenge/claude-kb (public)
- **Raw URL pattern:** `https://raw.githubusercontent.com/LoserChallenge/claude-kb/main/<path>`

## How to use it

1. **Read the index first.** `README.md` at the repo root lists every doc, what it covers, and
   where it is. That file is the authoritative index. This skill deliberately does not repeat it,
   so the two cannot drift apart.

2. **Open the doc the index points to — the section you need, not the whole file.** These docs are
   long by design. Each one carries its own jump table in the README entry.

3. **If the index has no match, search before giving up.** The index can lag a doc that was just
   added:

   ```
   Grep pattern="<term>" path="D:\cc-tech-support\claude-kb\docs" output_mode="content"
   ```

4. **If nothing in the repo covers it, say so plainly** and answer the normal way. Never guess at
   what a doc "probably" says, and never cite one you have not opened.

When a subject looks adjacent to the library but you are not certain it is covered, flag it and
ask — "there may be a doc on this, want me to check?" — rather than skipping silently. A loose
offer costs one line; a missed doc costs an answer given from memory.

## Reading these docs

Every doc here is a compiled snapshot, not live truth. These rules apply to all of them:

- **Carry the source tags.** Claims are tagged to the source they came from. When quoting a doc,
  carry the tag with the claim.
- **`[UNVERIFIED]` is not evidence.** Say a claim is unverified rather than passing it on as fact.
- **Check the compile date.** Each doc states when it was compiled and what it was verified
  against. Re-verify anything version-sensitive against the live source before relying on it.

## Adding a doc

Put the file in `docs/`, add an entry to `README.md`, commit, push. Nothing else changes — this
skill reads the index at run time, so it picks up new docs without being edited.

If the new doc covers a subject well outside what the `description` above names, add that subject
to the description too, or the skill will not trigger on it unprompted.
