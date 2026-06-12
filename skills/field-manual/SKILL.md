---
name: field-manual
description: >-
  Generate and maintain a single-file interactive HTML "field manual" that is
  the user's UI for the agent on a project — decision/consideration sections
  plus a task board with owner badges, per-task copy-prompt chips,
  browser-persisted checkmarks and notes, and a COPY STATUS FOR AGENT sync
  block. Triggers: "make a field manual", "project manual", "give me a UI for
  this project", "/field-manual", or a pasted block starting
  "## <DOC-ID> status sync".
---

# Field Manual

A field manual is one self-contained HTML file that serves as the user's control surface for a project the agent works on. The user reads decisions, checks off tasks, writes notes, and copies status blocks or per-task prompts back into agent sessions. The agent treats the file as shared state: it writes the task definitions; the user's checkmarks and notes live in their browser.

A filled example lives at `demo/demo.html` in this plugin's repository.

## Modes

Detect which mode applies before doing anything else.

### Mode A — Create

Trigger: user asks for a manual for a project.

1. **Research first.** A manual ships conclusions, not placeholders. Gather the facts the decision sections need (pricing, repo state, constraints) before writing HTML. Every claim that can go stale gets a verify-date and a source link in the footnote.
2. Copy `references/template.html` as the skeleton and follow `references/format.md` exactly — doc ID, design tokens, section order, TASKS schema, console contract. Format deviations break the user's muscle memory across manuals; do not improvise structure.
3. Fill `§` sections with the project's actual decisions (verdict cards for one-line answers, tables for comparisons), then the task board, then the agent-protocol section listing the standing rails for this project (what the agent never does without an explicit go).
4. Write the file to the project's folder (convention: `<project>/index.html`, or `manual.html` if index is taken) and open it in the user's browser.
5. If your environment has persistent memory, record the file path and doc ID as the project's control surface, with the reconcile rule below.

### Mode B — Reconcile

Trigger: user pastes a block starting `## <DOC-ID> status sync` (produced by the manual's COPY STATUS FOR AGENT button), or asks to "update the manual".

1. The paste is ground truth for task completion — trust it over session assumptions. `NOTE:` suffixes are user instructions; read every one.
2. Execute the next unchecked `agent`-owned tasks; `both` tasks the agent drives but confirms inputs with the user; `you` tasks are NEVER auto-executed (see rails).
3. Update the file: edit ONLY the `TASKS` array and any decision sections invalidated by new facts. Bump the Rev letter and date in the masthead. Never change the doc ID — it keys the user's saved state.
4. Task ids are append-only: never reuse or renumber. A dead task stays with its `detail` prefixed `SUPERSEDED:` rather than being deleted, unless the user asks for removal.
5. If the user keeps a root board (see Scaling up) and the reconcile changes a manual's scope or status (shipped, retired, renamed), update its registry row there in the same pass.

### Mode C — Single task

Trigger: user pastes a per-task prompt (the manual's ⌘ chips embed the doc ID and task id, e.g. "per DOC-ID manual task p2-3").

Execute that task only. Report; suggest the user tick the box. Do not start adjacent tasks.

## Scaling up — a root board for many manuals

Once the user has two or more manuals, offer one more manual that indexes them: a **root board** (suggested doc ID `ROOT-01`). It is an ordinary field manual — same template, same contract — whose job is making every other manual findable.

- A **registry** section holds a table of every live manual: doc ID, scope, link to the file, and its sync phrase (`<DOC-ID> status sync`). A manual missing from the registry is a bug.
- Its task board carries the cross-manual backlog: candidate manuals, system upkeep.
- Mirror the registry somewhere the user already looks — a browser bookmarks folder works well, root board first.
- **Birth checklist:** a new manual ships its file, a registry row in the root board (bump the root board's Rev letter and date), the bookmark, and a memory entry in one pass — creation isn't done until all four are. If editing the browser's bookmarks requires the browser to be closed (Chromium-family `Bookmarks` JSON does), do the bookmark step BEFORE opening the new manual.
- Retiring or renaming a manual reverses the same steps and needs an explicit user go.

## Rails (all modes)

- Owner `you` tasks are decisions or credentials — never do them, never simulate them done.
- Destructive or outward-facing actions (DNS, deletes, sends, purchases) require their explicit go-task to be checked in a status sync or stated by the user in chat.
- The agent never writes to the user's checkbox/note state; it lives in localStorage, not the file. File updates must keep task `id`s stable so that state survives.
- Never delete a manual, a root-board registry row, or its bookmark without an explicit user go; doc IDs are immutable forever.

## Format contract

The full normative spec — design tokens, TASKS schema, status-sync grammar, console behavior — is in `references/format.md`. Read it before writing or editing any manual.
