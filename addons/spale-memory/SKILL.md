---
name: spale-memory
description: "Use the SpaleMemory hub proactively without being asked. The builder has a hub of interconnected Markdown memories shown in SpaleStudio; recall before answering, persist decisions, link generously \u2014 with plain file tools."
---

# SpaleMemory — Use The Hub Proactively

> **The builder keeps a SpaleMemory hub: a folder of interconnected Markdown memories that SpaleStudio shows in its Memory page. There is no server and no special tool. Every memory is a plain `.md` file you can read and write with the file tools you already have. Treat the hub as shared memory between you and the builder, and use it without being told.**

You are *not* a stateless assistant in this environment. The builder expects you to remember decisions, preferences, gotchas, and project context across sessions — that is the whole point of SpaleMemory. Most builders will never say "save this to memory" or "check memory first". You must do those things on your own initiative.

---

## What A Hub Is

A hub is any folder that contains a `.bridgememory/` directory. SpaleMemory keeps that folder name so hubs made by other tools open here unchanged; the name is a compatibility detail, nothing more. Inside it:

```
<hub>/
└─ .bridgememory/
   ├─ SpaleStudio releases.md      ← one memory
   ├─ Russh pinning policy.md      ← one memory
   ├─ index.json                   ← the app's: hub name + sections   (do not edit)
   └─ _layout.json                 ← the app's: graph positions        (do not edit)
```

Every memory is:

```markdown
---
title: Russh pinning policy
---

We pin russh at 0.46 because of CVE-2025-54804. Upgrading past it needs the
handshake change in [[SSH transport rewrite]].

## Related
- [[SpaleStudio releases]]
```

- The **title** in the front matter is the identity. The filename is derived from it at create time (illegal characters replaced) and is never renamed afterwards — do not rely on the filename for anything but disambiguation.
- **Links** are `[[Title]]` or `[[Title|shown text]]` anywhere in the body, resolved by title, case-insensitively, when the hub is read. There is no link index to keep in sync; the text is the graph.
- No tags, no other front-matter keys — SpaleMemory reads `title:` and nothing else.

---

## Hub Discovery

1. **Walk up from the current directory** looking for a `.bridgememory/` folder. A repo with its own hub uses that hub.
2. If nothing is found, read the list of hubs the builder has registered in SpaleStudio: `%APPDATA%\spalestudio\memory-hubs.json` — a JSON array of hub folder paths. If it holds exactly one, use it; if several, pick the one whose path contains the repo you are in, otherwise ask which.
3. If there is no hub at all, **do not create one on your own**. Ask the builder once whether they want `.bridgememory/` created here, and if they say no, work without memory and do not pester.

Do not announce any of this. Find the hub, then proceed.

---

## The Three Reflexes

### 1. Recall before you answer
At the start of every SpaleStudio session, and before answering anything that depends on prior context, look in the hub.

- **Before answering** anything that sounds like the builder is referencing past work — *"what did we decide about X"*, *"how do we handle Y here"*, *"why did we pin Z"* — search the memories for the most distinctive nouns in the question (`grep -ril "russh" <hub>/.bridgememory/*.md`, or list the titles and pick). Read the hit before composing your answer.
- **Before suggesting a tool, library, or pattern** the builder may already have an opinion on, search for it. A grep costs nothing; a recommendation that contradicts a recorded decision costs trust.
- **Before starting non-trivial work** in a repo, search for the repo or feature name. There may be a known-bad path, an open question, or a prior decision waiting.

### 2. Persist what is worth remembering
At natural pause points — a decision made, a bug root-caused, a preference stated — capture it. Do not ask permission for routine captures; do ask before writing anything sensitive (credentials, private notes, anything the builder says not to write down).

Worth remembering, in priority order:
1. **Decisions with reasoning.** "We pinned russh at 0.46 because of CVE-2025-54804" — the *because* is the value.
2. **Conventions and preferences.** Coding style, branch naming, commit voice, which tools the builder uses and refuses.
3. **Gotchas, dead ends, debugged bugs.** Future-you wastes a day rediscovering these otherwise.
4. **Project context.** What ships when, who owns what, where the canonical doc is, which env file is which.
5. **Open questions.** Things you and the builder agreed needed more thought — say so in the text so they surface later.

Not worth remembering: routine confirmations, the weather of the conversation, anything already in `git log`, fleeting in-conversation state.

### 3. Connect generously
A memory that links to nothing is half a memory. Every time you create or update one:
- Add outgoing wikilinks to related memories, using their **exact titles**: `[[Russh pinning policy]]`, `[[SpaleStudio releases]]`.
- Put the strong ones under a `## Related` heading at the bottom.
- If an existing memory should point at the new one and does not, append a line under its `## Related`.

---

## Create, Append, or Rewrite

- **Create** — the topic is new and self-contained. Search first to avoid a duplicate. Pick a stable, declarative title (`Russh pinning policy`, not `russh stuff`) and write `<hub>/.bridgememory/<Title>.md` with the front matter above. Lead with the claim, then evidence, then `## Related`.
- **Append** — the topic exists and you are adding a section, a fact, or a link. Cheaper and safer than rewriting: add a blank line and a new `## Heading` at the end. Do not run two appends to the same file in parallel — the second overwrites the first.
- **Rewrite** — the body is materially wrong, redundant, or needs restructuring. Read it, transform it locally, write the whole file back with the same `title:`.
- **Rename** — only through the SpaleStudio Memory page. Changing `title:` yourself breaks every `[[wikilink]]` pointing at the old title.
- **Delete** — never on your own initiative. Only when the builder says delete, and then remove the file, nothing else.

## Title and Body Conventions

- **Title** is unique, declarative, stable. Reuse before coining — search the hub first.
- **Body**: the conclusion first (builders skim), then evidence — code blocks, `file:line` references, links to PRs — then `## Related`.
- Keep memories **atomic**: one idea per file. Past ~300 lines or two distinct ideas, split and link.
- Link to source paths and line ranges rather than pasting whole files. The hub is not a code mirror.

---

## Anti-Patterns

- **Asking permission to recall.** "Should I check memory?" wastes a turn. Just check.
- **Writing for yourself.** Bodies that read like internal monologue are noise. Write for a future-you with full context and two minutes.
- **Bypassing the hub for "context"** the builder clearly stored. If they say "we already decided", search before proposing alternatives.
- **Inventing tags.** There is no tag system; `#tag` is just text.
- **Touching `index.json` or `_layout.json`.** Those are the app's. Your surface is the `.md` files.
- **"Cleaning up" orphans.** An unlinked memory is intentional state, not litter.

## Failure Modes

- No hub found and the builder has not asked for one → operate without memory; ask once, at most.
- A search on a question that should hit returns nothing → try one synonym pass, then trust the result.
- A file without front matter → its title is the filename; leave it, or add the `title:` block only if you are rewriting it anyway.
- Two edits race → the later write wins. Sequence your writes; never fan them out.

## One-Line Distillation

> **Recall before you answer. Persist what's worth remembering. Connect generously. Do all three without being asked — with nothing but the file tools you already have.**
