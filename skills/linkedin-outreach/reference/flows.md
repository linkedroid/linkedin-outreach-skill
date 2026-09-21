# Flow shapes

## Contents
- Prefer a template
- The shapes that work
- Building a custom flow
- Rules the validator enforces

## Prefer a template

`campaigns.create` with a template is already laid out and proven. Build a
custom flow only when the user needs something no template does — then read the
catalogue with `campaigns.list_node_types` rather than guessing at node names
and ports.

| Template | Shape | Degree |
|---|---|---|
| `warm_outreach` | connect → wait → message | any |
| `connection_followup` | connect → wait → message → follow-up | any |
| `sales_outreach` | visit → connect → wait → message sequence | any |
| `content_engagement` | engage → connect → message | any |
| `message_blast` | message | **1st only** |
| `re_engagement` | message → wait → follow-up | **1st only** |

Message-first templates fail on 2nd and 3rd-degree prospects. The send does not
queue and retry — it fails.

## The shapes that work

**Degree-gated open** — the default for a mixed audience:

```
degree_filter
  ├─ 1st  → send_message
  ├─ 2nd  → send_connection → delay → condition(connected) → send_message
  └─ 3rd  → send_connection → delay → condition(connected) → send_message
```

**Warm before asking** — when the account is new or the audience is valuable:

```
profile_visit → delay 2d → engage → delay 2d → send_connection
```

**Stop on success** — so a campaign does not keep messaging someone who replied:

```
condition(replied)
  ├─ yes → goal "Replied" (stopCampaign: true)
  └─ no  → delay → follow-up
```

Leaving a port unconnected ends the sequence for prospects who take it. That is
usually what you want; it is not an error.

## Building a custom flow

1. `campaigns.list_node_types` — read purposes, ports and notes
2. Lay the sequence out top to bottom, roughly 130px apart
3. `campaigns.set_flow` — it validates and refuses anything that would silently
   fail
4. `campaigns.status` — lists message nodes still needing content
5. `campaigns.set_message` for each

## Rules the validator enforces

- Exactly one `start` node, with no incoming edges
- Each output port takes one edge — branch with `condition` or `degree_filter`
- Every node needs a `position`
- Edges are `{ id, source, sourcePort, target }`; `sourcePort` defaults to `out`

And one it does not enforce, that matters anyway: **put a delay between every
pair of actions.** Back-to-back contact reads as automation to the person
receiving it.
