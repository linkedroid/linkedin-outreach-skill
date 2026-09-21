# `campaigns_get` — MCP tool spec

Read a campaign back: its flow and the actual text of every message.

## Contents
- Why this is needed
- The tool
- Response shape
- Behaviour notes
- What it deliberately does not do

## Why this is needed

Today nothing can show a user what a campaign will say. `campaigns_status`
returns node content only for nodes that are **missing** content
(`needsContent`), and `campaigns_list_node_types` returns the generic
catalogue, not this campaign's configured flow.

The consequence is not cosmetic. The rule that nothing sends without the user
seeing the exact content is only satisfiable for a campaign the assistant wrote
itself, in the same session. For a campaign created last week, or in the web
UI, or by a colleague, an assistant asked "what will this send?" has no way to
answer — and an assistant asked to start it cannot show what it is starting.

Found by running a baseline evaluation: an assistant with no skill loaded
diagnosed five draft campaigns correctly, then reported it could not tell the
user what either of the ready ones would actually say.

## The tool

```
campaigns_get(campaignId: string) -> Campaign
```

Read-only. Touches nothing.

## Response shape

```jsonc
{
  "campaignId": "drip_1789241519031_306",
  "name": "Q4 Pipeline - SaaS Founders Warm Outreach",
  "type": "drip",
  "state": "draft",              // draft | running | paused | completed
  "audience": {
    "tags": ["Q4 Pipeline"],
    "excludeTags": [],
    "excludeContacted": true,
    "audienceLimit": 10
  },
  "schedule": { "enabled": false },
  "nodes": [
    {
      "id": "n2",
      "type": "send_connection",
      "position": { "x": 0, "y": 130 },
      "data": { "useTemplate": true, "noteMessage": "Hi {{firstName}}, ..." },
      "content": {                       // present on message-bearing nodes
        "text": "Hi {{firstName}}, ...",
        "characters": 283,
        "limit": 300,                    // null where the platform imposes none
        "variables": ["firstName"],
        "aiBlocks": [
          { "label": "Why", "prompt": "one line on why this person...", "valid": true }
        ]
      }
    }
  ],
  "edges": [
    { "id": "e1", "source": "n1", "sourcePort": "out", "target": "n2" }
  ],
  "needsContent": ["n5"]         // same field campaigns_status returns
}
```

## Behaviour notes

**Return the text verbatim.** Unrendered — `{{firstName}}` and
`[[ai:Label::prompt]]` exactly as stored. The caller is showing a human what
will be sent, and a pre-rendered sample for one prospect hides what everyone
else receives.

**Include `characters` and `limit` per message node.** The 300-character
connection-note cap is silently truncating, so the one place it can be caught
is here, before the campaign runs.

**Validate AI blocks and say so.** An `[[ai:…]]` block without a `Label::` is
silently discarded at send time and the message goes out with a hole in it.
`aiBlocks[].valid: false` turns a silent failure into a visible one. This is
the highest-value field in the response.

**Keep `needsContent` consistent with `campaigns_status`.** Two tools reporting
the same condition differently is worse than one tool reporting it.

**Errors:** `CAMPAIGN_NOT_FOUND`. Scope to the authenticated user like every
other tool.

## What it deliberately does not do

**No rendering per prospect.** Tempting — "show me what Jane receives" — but it
would mean running AI generation at read time, which is slow, costs money, and
produces something nobody actually sends.

**No editing.** Reading and writing stay separate. `campaigns_set_message` and
`campaigns_set_flow` already own the write path.
