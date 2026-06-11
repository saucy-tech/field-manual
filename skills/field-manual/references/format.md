# Field Manual — Format Contract

Normative spec for every manual this skill produces or edits. `template.html` in this directory is the skeleton; `demo/demo.html` in the repository is a filled example.

## 1. File

- One self-contained HTML file. No build step. Only external dependency: Google Fonts.
- Location: the project's folder — `<project>/index.html` by convention (`manual.html` if index is taken).
- Must render correctly from `file://` and survive offline except for fonts.

## 2. Identity

- Masthead carries: `Doc <PREFIX>-<NN> · Rev <letter> · <YYYY-MM-DD> · <owner>` plus the title (`Fraunces 900` with an italic-300 accent word) and a rotated stamp chip.
- **Doc ID** (`ACME-MIG-01` style) doubles as the localStorage key. It is immutable after creation — changing it orphans the user's saved checkmarks and notes.
- Agent edits bump the Rev letter and date; the user never edits the file.

## 3. Design system

Locked structure; only the accent hue may vary per project.

| Token | Value | Role |
|---|---|---|
| `--paper` | `#f4f0e6` | page background (with ruled-paper repeating-linear-gradient lines) |
| `--ink` | `#1c1a16` | text, borders, phase headers |
| `--accent` | `#c8501e` default; project may swap hue | stamp, section `§` numbers, links, owner-you badge |
| `--ok` / `--wait` | `#2e6b3f` / `#8a6d1c` | done states / joint-owner badge |
| progress bar | `#f6821f` or accent | console bar fill |

Fonts: **Fraunces** (display: 900 headings, 300-italic accents), **Public Sans** (body), **IBM Plex Mono** (doc id, meta lines, chips, tables' headers). Do not substitute.

Recurring furniture: double-rule under masthead, `§n` section numbers, verdict cards (bordered, inset rule, mono question / display answer / small why), tables with mono uppercase headers, print stylesheet hiding nav/console.

## 4. Section order

1. **How-this-works box** (dashed accent border) — 4-step usage: check tasks, copy status, per-task chips, agent updates file.
2. `§1..§n` decision/consideration sections — verdict cards for one-line answers, tables for comparisons, prose ≤ 72ch.
3. **§ Task board** — phases rendered from the `TASKS` array.
4. **§ Agent protocol** — standing rails for this project; file path + memory key.
5. Footnote — sources with the date pricing/facts were verified.

## 5. TASKS schema

The single `const TASKS = [...]` array is the only thing the agent edits during reconcile.

```js
{ phase: "P2 · Secrets + first deploys",   // "P<n> · Title"
  items: [
    { id: "p2-1",          // stable, lowercase, append-only, never reused
      owner: "you" | "agent" | "both",
      title: "...",        // imperative, one line
      meta: "...",         // mono subline: URLs, env var names, constraints
      detail: "...",       // 1–2 sentences: why / what success looks like
      blockedBy: ["p1-1"], // optional; task renders dimmed until those ids are checked
      prompt: "..."        // self-contained agent instruction embedding doc id +
                           // task id + absolute paths; "" when owner is you-only
    }
  ]}
```

Owner semantics: `you` = decision/credential, agent never executes; `agent` = paste the prompt, agent does it solo; `both` = agent drives, user supplies inputs/approval.

## 6. State separation

- User state (checkbox + note per task id) persists to `localStorage[docId]` as `{ "<id>": { done, note } }`.
- The file never contains user state; agent file updates therefore can't clobber it.
- Stable ids are the contract. Removing an id silently abandons its state — prefer `SUPERSEDED:` in `detail`.

## 7. Console contract

Sticky bottom bar: label, progress bar + `done/total` counter, **COPY STATUS FOR AGENT**, ghost **reset** (confirm dialog). Per-task chips: `⌘ copy prompt for agent` (clipboard) and `✎ note` (toggles the persisted textarea). Toast confirms copies.

Status block grammar (what Mode B parses — keep exact):

```
## <DOC-ID> status sync · YYYY-MM-DD

### P1 · <phase title>
- [x] P1-1 <title> — NOTE: <user note, only if present>
- [ ] P1-2 <title>

_Paste from <file path> — reconcile and continue._
```

## 8. Polish features (template includes; keep working when editing)

- Collapsible phases (click phase header; chevron rotates; collapsed state is per-browser, not synced).
- `blockedBy` dimming — blocked tasks at reduced opacity with a `⛔ waits on <ids>` meta suffix; checkbox disabled until unblocked.
- Print stylesheet.

Deliberately out of scope (do not add without the user asking): server sync, file-writing from the browser, frameworks, bundlers, analytics.
