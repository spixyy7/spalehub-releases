---
name: spale-obsidian
description: "Operate an Obsidian vault as an agent \u2014 vault structure, frontmatter, wikilinks, daily notes, and the three integration paths (filesystem, URI scheme, Local REST API)."
---

# SpaleObsidian — Vault Operator Skill

> **An Obsidian vault is a folder of plain Markdown files. Treat it as a structured filesystem, not a magical app. Every operation an agent needs — read, write, link, tag, search, schedule — has a deterministic path.**

Obsidian is the highest-leverage knowledge tool an agent can drive: notes are plain `.md` files on disk, links are stable, metadata is YAML, and the storage layer requires zero API calls. This skill teaches you the conventions to follow and the three ways to talk to a vault.

---

## Phase 1: Locate the Vault

Before touching anything, **confirm the vault root**. A vault is any folder containing a `.obsidian/` subdirectory — that's the only structural marker.

```bash
ls -la "$VAULT_PATH/.obsidian"      # confirm it's a vault
```

**Common vault locations:**
- macOS: `~/Documents/<VaultName>`, `~/Obsidian/<VaultName>`, iCloud Drive
- Linux: `~/Documents/<VaultName>`, `~/Notes`
- Windows: `%USERPROFILE%\Documents\<VaultName>`

**Never assume the vault name.** Ask the builder, or look up the active vault from `~/Library/Application Support/obsidian/obsidian.json` (macOS) / `%APPDATA%\obsidian\obsidian.json` (Windows) / `~/.config/obsidian/obsidian.json` (Linux). That JSON file contains `{ "vaults": { "<id>": { "path": "...", "open": true } } }`.

If the path doesn't exist or has no `.obsidian/` folder, **stop and ask the builder** — don't create one silently.

---

## Phase 2: Vault Structure Conventions

Structure is the API between the builder and you. Adopt one of these patterns and **stay consistent**:

### PARA-style (recommended for general knowledge work)

```
VaultRoot/
├── 00_Inbox/          # Unsorted captures — process and move out
├── 10_Projects/       # Active work with deadlines
├── 20_Areas/          # Ongoing responsibilities (health, finances, role)
├── 30_Resources/      # Topics of interest, reference material
├── 40_Archive/        # Completed / inactive items
├── 90_Daily/          # Daily notes (YYYY/MM/YYYY-MM-DD.md)
├── 99_Templates/      # Note templates
├── _attachments/      # Pasted images, PDFs, audio
└── .obsidian/         # Vault config — do not write here
```

### Agent-augmented layout

When the vault is co-managed with an AI agent, **isolate agent-managed content** from human notes:

```
VaultRoot/
├── _agent/
│   ├── memory/        # working / episodic / semantic notes
│   ├── heartbeat/     # scheduled outputs (daily summaries, weekly reviews)
│   ├── context/       # active context windows
│   └── runs/          # transcripts and results from agent runs
├── ...                # human-managed PARA folders above
```

**Rules:**
- Use **numeric prefixes** (`00_`, `10_`) so folders sort consistently across OSes.
- Avoid spaces in folder names — use `_` or `-`. (Spaces work but break shell commands.)
- Never write inside `.obsidian/` — it's the app's config.
- Never write inside `_attachments/` directly — Obsidian places files there when paste/drag-dropped.

---

## Phase 3: Note Anatomy

Every well-formed note has three sections, in this order:

```markdown
---
title: Trip to Paris
created: 2026-05-04
tags: [travel, planning]
status: in-progress
related: ["[[Travel hub]]", "[[Q2 goals]]"]
---

# Trip to Paris

Definitive opening sentence — what is this note about, in one line.

## Section heading
Body content with [[wikilinks]] and #inline-tags.
```

### Frontmatter (YAML between `---` fences)
- **First three lines must be `---` … `---`.** No blank line before the opening fence.
- Property names are lowercase, kebab-case for multi-word: `due-date`, `reading-status`.
- Internal links inside frontmatter **must be quoted strings**: `related: "[[Note Name]]"` or list form `related: ["[[A]]", "[[B]]"]`.
- Date format: `YYYY-MM-DD` (date only) or full ISO 8601 (`2026-05-04T09:00:00Z`) for date+time.
- Tags can live in frontmatter (`tags: [travel, planning]`) **or** inline (`#travel`) — Obsidian merges them. Pick one style per project; mixing creates dataview headaches.
- Multiline text uses `|` (literal block) or `>` (folded).

### Body
- Exactly **one H1** — the note's title (often duplicated from `title` frontmatter).
- Use H2 for primary sections, H3 for sub-sections. **Never skip levels.**
- First sentence is what gets surfaced in link previews and AI chunking — make it definitive.
- Avoid these characters in filenames: `# | ^ : / \ % "`. Obsidian rejects or escapes them.

---

## Phase 4: Wikilinks & Backlinks

Wikilinks are the core of Obsidian's value. Use them everywhere a concept is mentioned.

| Syntax | Renders as | When to use |
|---|---|---|
| `[[Note Name]]` | Link to "Note Name.md" | Standard internal link |
| `[[Note Name|Display]]` | Link with custom anchor text | Inline prose flow |
| `[[Note#Heading]]` | Link to a specific H2/H3 | Targeting a section |
| `[[Note#^block-id]]` | Link to a specific block | Targeting a paragraph |
| `[[#Heading]]` | Link inside the same note | Table of contents |
| `![[Note]]` | **Embed** the note's content inline | Transclude another note |
| `![[image.png]]` | Embed an image | Inline media |
| `![[Note#Heading]]` | Embed a specific section | Reuse a heading's content |

**Rules:**
- Obsidian resolves wikilinks **by filename**, not full path. Two notes with the same name break linking — name notes uniquely or use full paths.
- Linking to a non-existent note creates a *placeholder* (italicized in the UI). Useful for "stub" notes you'll fill in later.
- **Aliases** (`aliases: [Foo, Bar]` in frontmatter) let `[[Bar]]` resolve to a note titled `Foo`.
- Backlinks are automatic — every note shows what links to it. Use this as your "mentioned in" graph.

**When you create a new note, immediately link it to at least one existing note.** Orphan notes are dead weight.

---

## Phase 5: Tags vs. Folders vs. Properties

Three overlapping organization systems. Pick the right one per concern:

| Use | For |
|---|---|
| **Folder** | Where a note lives (state-based: Active / Archived / Inbox) |
| **Tag** | What a note *is about* (cross-cutting topics: `#meeting`, `#book`, `#bug`) |
| **Property** | Structured metadata (status, due-date, score, author) |

**Rules:**
- Tag format: `#topic` inline or `tags: [topic]` in frontmatter. Use `#parent/child` for hierarchy. **Do not use spaces** — use `-` or `_` (`#side-project`, not `#side project`).
- Don't tag what's obvious from the folder. If everything in `10_Projects/` is a project, `#project` adds nothing.
- Properties are the right place for any field you'll **filter, sort, or query** later.

---

## Phase 6: Daily & Periodic Notes

Daily notes are the workhorse of an Obsidian vault. They give every agent a predictable place to log activity.

**Standard layout:** `90_Daily/2026/05/2026-05-04.md`

**Daily note template:**
```markdown
---
date: 2026-05-04
type: daily
---

# 2026-05-04

## Focus
- Top thing to ship today.

## Log
- 09:00 — Started X.
- 11:30 — Hit Y, decision: Z.

## Captures
- [[Idea seed]]
- Link to [[Meeting with N]].

## Tomorrow
- [ ] Carry-over task.
```

**Periodic notes** (weekly, monthly, quarterly, yearly) follow the same pattern with `type: weekly`, `type: monthly`, etc., and a corresponding folder.

When writing automation that runs daily, **always append** to the existing daily note rather than overwriting. Use the PATCH-by-heading pattern (see Phase 9) so you preserve human-written content.

---

## Phase 7: Templates

Templates live in `99_Templates/` and use either Obsidian's built-in template syntax (`{{date}}`, `{{title}}`) or the **Templater** plugin (`<% tp.date.now() %>`).

**Built-in placeholders:**
- `{{date}}` / `{{date:YYYY-MM-DD}}`
- `{{time}}` / `{{time:HH:mm}}`
- `{{title}}` — current filename

**Agent pattern:** when creating a new note, read the matching template from `99_Templates/`, expand placeholders, then write the file. Don't reinvent the structure each time.

---

## Phase 8: Three Integration Paths (pick the right one)

Obsidian gives you three ways to talk to a vault. **Choose by capability:**

| Path | When to use | Requires |
|---|---|---|
| **Direct filesystem** | Default for any read/write. Fastest, no Obsidian process needed. | Vault path |
| **Obsidian URI scheme** | Triggering UI actions (open a note, run a command, switch vault) | Obsidian app installed |
| **Local REST API plugin** | Live operations on a running vault — search, append-by-heading, list commands, periodic notes | Obsidian app **running** + plugin enabled |

### 8.1 Direct filesystem (the default)

Any agent that can read/write files can read/write notes:

```bash
# Read a note
cat "$VAULT/10_Projects/Trip to Paris.md"

# Create a new note
cat > "$VAULT/00_Inbox/$(date +%Y-%m-%d)-capture.md" <<'EOF'
---
created: 2026-05-04
tags: [inbox]
---

# Capture
Body.
EOF

# Search across the vault
grep -rl --include="*.md" "search term" "$VAULT"
```

**When the Obsidian app is running**, it auto-detects filesystem changes and reloads. **No need to "save" via the app.**

**Rules:**
- Always write atomically: write to a temp file in the same directory, then `mv` into place. Prevents partial files if the agent dies mid-write.
- Use UTF-8. No BOM.
- Use `\n` line endings (LF), not CRLF — even on Windows. Obsidian normalizes either way, but git diffs stay clean with LF.
- Don't write to `.obsidian/` or `.trash/`.

### 8.2 Obsidian URI scheme

For any action that should happen in the **app UI** (open a note, switch vaults, trigger a command), use the URI scheme:

| URI | Effect |
|---|---|
| `obsidian://open?vault=MyVault&file=10_Projects%2FTrip` | Open a note |
| `obsidian://new?vault=MyVault&name=Quick%20note&content=Body` | Create + open a note |
| `obsidian://daily?vault=MyVault` | Open today's daily note |
| `obsidian://search?vault=MyVault&query=tag%3A%23book` | Open search with a query |
| `obsidian://advanced-uri?vault=MyVault&commandid=editor:save-file` | Run any registered command (requires Advanced URI plugin) |

**Always URL-encode values.** Spaces → `%20`, slashes in paths → `%2F`, hashes → `%23`.

```bash
# macOS
open "obsidian://open?vault=MyVault&file=10_Projects%2FTrip"

# Linux
xdg-open "obsidian://..."

# Windows
start "" "obsidian://..."
```

URIs **do not return data** — they're fire-and-forget actions. Use the REST API or filesystem for anything you need to read back.

### 8.3 Local REST API plugin (live operations)

The [Local REST API plugin](https://github.com/coddingtonbear/obsidian-local-rest-api) exposes the running Obsidian instance over HTTP. Install it once via Community Plugins, copy the API key from Settings → Local REST API.

**Defaults:**
- HTTPS: `https://127.0.0.1:27124` (self-signed cert — use `-k` with curl or trust `/obsidian-local-rest-api-certificate.crt`)
- HTTP: `http://127.0.0.1:27123` (only if explicitly enabled)
- All endpoints (except `/`) require `Authorization: Bearer <api-key>`

**Endpoint cheat-sheet:**

| Endpoint | Method | Use |
|---|---|---|
| `/` | GET | Status check, server info |
| `/vault/{path}` | GET / PUT / POST / PATCH / DELETE | Read / overwrite / append / patch / delete a file |
| `/vault/{dir}/` | GET | List files in a directory (trailing slash matters) |
| `/active/` | GET / PUT / POST / PATCH / DELETE | Operate on whatever note is open in the UI |
| `/periodic/{period}/` | GET / PUT / POST / PATCH / DELETE | Today's daily/weekly/monthly/quarterly/yearly note |
| `/periodic/{period}/{yyyy}/{mm}/{dd}/` | … | A specific dated periodic note |
| `/search/simple/?query=...` | POST | Full-text search |
| `/search/` | POST | Dataview DQL or JsonLogic queries (see plugin docs) |
| `/commands/` | GET / POST | List all Obsidian commands; execute by id |
| `/tags/` | GET | All tags + usage counts |
| `/open/{filename}` | POST | Reveal a file in the UI |

**Examples:**

```bash
KEY="<your-api-key>"
BASE="https://127.0.0.1:27124"

# Status
curl -ks "$BASE/"

# Read a note
curl -ks -H "Authorization: Bearer $KEY" \
  "$BASE/vault/10_Projects/Trip%20to%20Paris.md"

# Read a note as JSON (frontmatter parsed, tags extracted, etc.)
curl -ks -H "Authorization: Bearer $KEY" \
  -H "Accept: application/vnd.olrapi.note+json" \
  "$BASE/vault/10_Projects/Trip%20to%20Paris.md"

# Create or overwrite a note
curl -ks -X PUT -H "Authorization: Bearer $KEY" \
  -H "Content-Type: text/markdown" \
  --data-binary @new-note.md \
  "$BASE/vault/00_Inbox/new-note.md"

# Append to a note
curl -ks -X POST -H "Authorization: Bearer $KEY" \
  -H "Content-Type: text/markdown" \
  --data-binary "Appended line\n" \
  "$BASE/vault/90_Daily/2026/05/2026-05-04.md"

# Append under a specific heading (preserves the rest of the note!)
curl -ks -X PATCH -H "Authorization: Bearer $KEY" \
  -H "Operation: append" \
  -H "Target-Type: heading" \
  -H "Target: Log" \
  -H "Create-Target-If-Missing: true" \
  -H "Content-Type: text/markdown" \
  --data-binary "- 14:00 — Shipped X." \
  "$BASE/periodic/daily/"

# Update a frontmatter field
curl -ks -X PATCH -H "Authorization: Bearer $KEY" \
  -H "Operation: replace" \
  -H "Target-Type: frontmatter" \
  -H "Target: status" \
  -H "Content-Type: application/json" \
  --data '"complete"' \
  "$BASE/vault/10_Projects/Trip%20to%20Paris.md"

# Search
curl -ks -X POST -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  --data '{"query": "Paris", "contextLength": 100}' \
  "$BASE/search/simple/"

# Run a command (e.g. open today's daily note)
curl -ks -X POST -H "Authorization: Bearer $KEY" \
  "$BASE/commands/daily-notes/"
```

**PATCH operations** are the most powerful endpoint — they let you target a heading, block reference, or frontmatter field and `append` / `prepend` / `replace` content surgically. Use them whenever you need to update part of a note without overwriting the whole thing.

---

## Phase 9: Search Patterns

Obsidian's search syntax (also accepted by `/search/simple/`):

| Operator | Example | Match |
|---|---|---|
| Plain text | `paris` | Notes containing "paris" |
| `tag:` | `tag:#book` | Notes with that tag |
| `file:` | `file:Trip` | Notes whose filename matches |
| `path:` | `path:10_Projects` | Notes inside that folder |
| `line:` | `line:(todo paris)` | Both terms on the same line |
| `section:` | `section:(travel paris)` | Both terms in the same section |
| `/regex/` | `/^# Trip/` | Regex match |
| `-term` | `paris -archive` | Exclude |
| `"phrase"` | `"trip to paris"` | Exact phrase |
| `OR` | `paris OR london` | Either term |

For richer queries, use **Dataview** (a separate plugin) — it can query frontmatter, build tables, and aggregate. Example:

```dataview
TABLE status, due-date
FROM "10_Projects"
WHERE status != "complete"
SORT due-date ASC
```

Dataview blocks live inside ```dataview … ``` fences in the note body.

---

## Phase 10: Standard Operating Procedures

### SOP 1: Capture an idea
1. Locate vault, confirm `00_Inbox/` exists.
2. Filename: `YYYY-MM-DD-short-slug.md`.
3. Frontmatter with `created`, `tags: [inbox]`.
4. One-sentence opening, then the capture.
5. Optionally append a wikilink in today's daily note's "Captures" section.

### SOP 2: Append a log entry to today's daily note
1. Compute today's path: `90_Daily/$(date +%Y/%m)/$(date +%Y-%m-%d).md`.
2. If file doesn't exist, create from `99_Templates/Daily.md`.
3. PATCH-append under the `## Log` heading via REST API, **or** filesystem-append with a timestamp prefix.
4. Never overwrite — always append.

### SOP 3: Create a project note
1. Filename in `10_Projects/`: `Project Name.md` (Title Case, spaces OK).
2. Frontmatter: `status: planning|in-progress|in-review|complete`, `due-date`, `tags`, optional `related`.
3. H1 matching filename. H2 sections: Goal, Scope, Tasks, Decisions, Notes.
4. Link from at least one existing note (project hub, area, or daily note).

### SOP 4: Move a note to Archive
1. `mv` the file from its current folder to `40_Archive/<original-folder>/`.
2. Optionally update frontmatter: `status: archived`, `archived: <date>`.
3. **Backlinks survive moves** — Obsidian updates them automatically when the app is running. If editing offline, run "Update internal links" command after.

### SOP 5: Bulk-tag or bulk-update notes
1. For each note, read frontmatter.
2. Add/modify the field via PATCH-frontmatter (REST API) or by rewriting the YAML block (filesystem).
3. **Never rewrite the body** — only the frontmatter block. This preserves the human's content.

---

## Phase 11: Hard Rules

1. **Never write inside `.obsidian/`** — that's the app's config. You will corrupt the vault.
2. **Never overwrite a note when you mean to append.** Use POST, PATCH, or filesystem append.
3. **Always quote internal links in YAML frontmatter** (`related: "[[Foo]]"`).
4. **Never use spaces in tags.** Use `-` or `_` (`#side-project`).
5. **Never assume the vault path** — discover it from `obsidian.json` or ask the builder.
6. **Always URL-encode** values in `obsidian://` URIs and REST API paths.
7. **Always include the trailing slash** when listing a directory via `/vault/dir/` — without it the API tries to read it as a file.
8. **Never commit the Local REST API key** anywhere — it grants full vault read/write.
9. **Never write outside the vault root**, even when given a relative path that looks like it stays inside (`../` traversal). Resolve and validate.
10. **Match the vault's existing conventions.** If the user already has `Inbox/` instead of `00_Inbox/`, follow that — don't impose PARA on a working vault.

---

## Phase 12: Common Mistakes

- ❌ Creating a note with the same filename as an existing one — breaks wikilink resolution silently.
- ❌ Using `PUT /vault/...` when you meant `POST` — PUT overwrites, POST appends.
- ❌ Writing CRLF line endings on Windows — works in Obsidian but trashes git diffs.
- ❌ Embedding huge files (`![[10mb-pdf.pdf]]`) inline — they bloat note rendering. Link instead.
- ❌ Building a deeply nested folder hierarchy (>3 levels) — you lose findability and Obsidian's flat-search advantage.
- ❌ Using frontmatter for content the human will never query — keeps notes readable and YAML lean.
- ❌ Triggering `obsidian://new?...` for a note you also want to read back — the URI doesn't return the new note. Read it via filesystem afterward.
- ❌ Forgetting that REST API endpoints require the **app to be running** — gracefully fall back to filesystem if the request times out.

---

## Phase 13: When You're Unsure

1. **"Where is the vault?"** → Check `obsidian.json`; if missing, ask the builder.
2. **"Should this be a new note or an addition to an existing one?"** → If the topic already has a note, append/embed-link. Otherwise create + cross-link.
3. **"Folder, tag, or property?"** → Folder = state, tag = topic, property = queryable field.
4. **"Filesystem, URI, or REST API?"** → Default filesystem. URI for UI-side actions. REST API for live PATCH/search/commands when Obsidian is running.
5. **"Is this safe to overwrite?"** → Default no. Always prefer append, PATCH-by-heading, or write-to-new-file.

---

## One-Line Distillation

> **A vault is plain Markdown on disk. Find it → respect its structure → write atomically → link generously → update surgically. The app is optional; the conventions are not.**
