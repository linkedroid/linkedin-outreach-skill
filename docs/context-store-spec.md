# Context store — MCP tool spec

Four tools that let a skill remember the user's business between sessions.

## Contents
- Why this is server-side
- The tools
- Reserved keys
- Behaviour notes
- What this deliberately does not do

## Why this is server-side

The obvious implementation is files in the skill directory. That does not work
here: the skill runs in whatever client the user has — claude.ai, Claude
Desktop, the Linkedroid panel — and most of those have no writable filesystem
the skill can rely on. Storage has to live where the MCP server already
authenticates the user.

This is the difference between a prompt and a product. Without it the skill
re-interrogates the user about their business every session, which is the
experience the product exists to replace.

## The tools

### `context_set`

Store one value. Replaces whatever was there.

```
context_set(key: string, value: string) -> {
  key: string,
  bytes: number,
  updatedAt: string   // ISO 8601
}
```

| Field | Rule |
|---|---|
| `key` | 1–64 chars, lowercase letters, digits, `.` and `_`. Dots namespace: `icp.primary` |
| `value` | UTF-8 text, max **16 KB**. Markdown expected, but the store does not parse it |

Errors: `KEY_INVALID`, `VALUE_TOO_LARGE`, `QUOTA_EXCEEDED`.

### `context_get`

Read values. Omit `keys` to get everything.

```
context_get(keys?: string[]) -> {
  items: [{ key: string, value: string, updatedAt: string }],
  missing: string[]     // keys asked for that do not exist
}
```

`missing` is explicit so a skill can tell "never set" from "set to empty" —
the difference between asking an onboarding question and not.

Omitting `keys` is capped at **64 KB total**; beyond that it returns the
smallest values first and lists the rest in `truncated`.

### `context_list`

Keys and sizes without the content — cheap enough to call at session start.

```
context_list() -> {
  items: [{ key: string, bytes: number, updatedAt: string }]
}
```

This is what makes progressive disclosure work: the skill sees `icp` exists and
was set three months ago, and decides whether to read it or re-confirm it,
without paying for the content first.

### `context_delete`

```
context_delete(key: string) -> { deleted: boolean }
```

`deleted: false` when the key was not there. Not an error — deleting something
absent is the state the caller wanted.

## Reserved keys

The store takes any key. These are the ones the skills use, and they should be
documented so third parties do not collide:

| Key | Written by | Holds |
|---|---|---|
| `icp` | onboarding-icp | who the user sells to, in prose |
| `offer` | onboarding-icp | what they sell and why it is worth a reply |
| `voice` | calibrating-voice | how the user writes: register, length, phrases to avoid |
| `anchors` | onboarding-icp | the qualification rubric, so scoring stays consistent |
| `do_not_contact` | any | people and companies never to approach |

`do_not_contact` earns its place commercially. Customers, competitors, a
founder's own investors — contacting the wrong person once costs more trust
than a whole campaign earns, and today nothing prevents it.

## Behaviour notes

**Scope to the authenticated user**, the same way every other Linkedroid tool
does. No cross-user reads, ever — this holds commercial strategy.

**Quota per user: 256 KB total.** Enough for the reserved keys many times over.
A skill that needs more than that is storing the wrong kind of thing.

**Return `updatedAt` everywhere.** Staleness is the main failure mode: an ICP
set before a pivot is worse than no ICP, because it is followed confidently.
A skill that can see the date can ask.

**No merge semantics.** `context_set` replaces. Read-modify-write belongs in
the caller, where the format is understood. A store that tries to merge
markdown will merge it wrongly.

## What this deliberately does not do

**No schema or validation.** The store holds text. Formats are the skills'
business, and baking one in means a tool release every time a skill's format
changes.

**No history or versioning.** Tempting, and not worth the complexity for v1.
If a user wants their old ICP back, they can write it again.

**No sharing between users.** Team-level context is a real future need — an
agency running outreach for several clients — but it needs a permission model
this does not have. Do it as a separate `workspace_*` namespace later rather
than bending this one.
