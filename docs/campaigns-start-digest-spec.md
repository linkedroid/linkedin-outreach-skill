# `campaigns_start` digest confirmation — MCP tool spec

Make "nothing sends until a human has seen it" a property of the system
rather than a sentence in a skill file.

## Contents
- The problem
- The flow
- Preview response
- Confirm call
- What the digest covers
- What this does and does not guarantee
- Why not a `force` flag

## The problem

Starting a campaign is the one irreversible action in the tool surface. A
connection request cannot be unsent; a first-touch message lands once. Every
other tool is recoverable.

The current protection is advisory: prose, in a skill file, asking the
assistant to show the user the messages first. Advisory protection fails in the
ordinary case, not the adversarial one — a long session, a compacted context, a
user who says "just go", a campaign written three turns ago. None of that is
misbehaviour, and all of it produces a send nobody reviewed.

Worse, the advice was until now unfollowable for any campaign the assistant did
not write itself, because nothing could read the messages back. That part is
fixed by `campaigns_get`; this spec fixes the rest.

## The flow

`campaigns_start` becomes two calls. The first previews and returns a digest;
the second starts and must present that digest back.

```
campaigns_start(campaignId)                      -> preview, does not start
campaigns_start(campaignId, confirm: "<digest>") -> starts, or refuses
```

The confirm call succeeds only when the digest still matches the campaign's
current state. Edit a message between the two calls and the digest no longer
matches: the start is refused and a fresh preview comes back. Approval is bound
to specific content, so approve-then-edit is not expressible.

This is a compare-and-swap, and it is the whole mechanism.

## Preview response

```jsonc
{
  "started": false,
  "preview": {
    "campaignId": "drip_1789241519031_306",
    "name": "Q4 Pipeline - SaaS Founders Warm Outreach",
    "audience": {
      "count": 10,
      "source": "tag: Q4 Pipeline",
      "excludeContacted": true,
      "sample": [                       // first 5, so the user can sanity-check targeting
        { "name": "...", "headline": "...", "degree": 2 }
      ]
    },
    "messages": [                       // every message node, verbatim, in flow order
      {
        "nodeId": "n2",
        "type": "send_connection",
        "text": "Hi {{firstName}}, ...",
        "characters": 283,
        "limit": 300,
        "aiBlocks": [ { "label": "Why", "valid": true } ]
      }
    ],
    "schedule": { "enabled": false, "warning": "SCHEDULE_DISABLED" },
    "firstActionAt": "immediately"
  },
  "blockers": [],                       // hard stops — start is impossible until cleared
  "warnings": ["SCHEDULE_DISABLED"],    // soft — start is allowed, user should know
  "digest": "sha256:9f2c...",
  "digestExpiresAt": "2026-09-21T18:45:00Z"
}
```

**Blockers** (start refused regardless of digest): `NEEDS_CONTENT` — a node has
no message; `EMPTY_AUDIENCE` — nothing resolves; `INVALID_AI_BLOCK` — an
`[[ai:…]]` block missing its `Label::`, which sends a message with a hole in
it; `OVER_CHARACTER_LIMIT` — a connection note past 300, which truncates
silently.

**Warnings** (start allowed): `SCHEDULE_DISABLED` — actions fire at whatever
hour the campaign is started rather than during business hours;
`LARGE_AUDIENCE` — the audience is big enough that the weekly invitation cap
will be the binding constraint; `PENDING_INVITES_HIGH` — stale pending
invitations are already consuming that cap.

## Confirm call

```
campaigns_start(campaignId, confirm: "sha256:9f2c...")
```

| Outcome | Response |
|---|---|
| Digest matches, no blockers | `{ "started": true, ... }` |
| Digest stale | `DIGEST_MISMATCH` + a fresh preview and digest, naming what changed |
| Digest expired | `DIGEST_EXPIRED` + a fresh preview |
| Blocker present | `CAMPAIGN_BLOCKED` + the blocker list |
| Digest absent | the preview — i.e. the first call |

Give `DIGEST_MISMATCH` a `changed` array (`["messages.n2", "audience.count"]`).
"Something changed, here it is again" is a dead end for an assistant trying to
explain itself to a user; "the connection note changed" is actionable.

TTL of 15 minutes. It stops an approval from being replayed into a later
session, where the context that produced it is gone.

## What the digest covers

A stable hash over: every message node's text, the flow's edges, the resolved
audience count and member ids, and the schedule. Not the campaign name, not
node positions, not `updatedAt` — cosmetic churn must not invalidate a real
approval, or the mechanism trains everyone to treat mismatches as noise.

Including resolved member ids is the deliberately strict choice. It means an
audience that grows from 10 to 400 between preview and confirm forces a second
look. That is the failure this is for.

## What this does and does not guarantee

Worth being exact, because the value of the mechanism is easy to overstate.

**It does** force the full content of what will send into the conversation
before a start is possible, put it there in the user's own transcript where
they can read it, bind the approval to that exact content, and make an
unreviewed start require a second deliberate call rather than falling out of a
single ambiguous instruction.

**It does not** verify that a human read anything. An assistant can call
preview and confirm back to back without surfacing either. No server-side
mechanism can tell the difference.

So this raises the floor, it does not close the hole. It converts "the model
was asked nicely" into "the model had to take a second, explicit, content-bound
action" — which survives context compaction, long sessions and vague
instructions, the three conditions under which the prose version actually
fails. That is a real improvement and it is a bounded one.

## Why not a `force` flag

Any bypass becomes the default path the moment one start is inconvenient, and
then the guarantee is gone everywhere while still appearing in the docs.

The web UI is a different matter: it has its own review screen, so the digest
requirement belongs at the MCP layer, not in the core start path. Same engine,
different door.
