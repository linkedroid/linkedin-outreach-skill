# `campaigns_start` digest confirmation — withdrawn

**Status: superseded. Do not implement.** Kept because the reasoning that
replaced it is more useful than the proposal was.

## What this proposed

A two-call handshake on `campaigns_start`: the first call returns a preview and
a content digest, the second must present that digest back, and a mismatch
refuses the start. The aim was to stop a campaign being started without anyone
seeing what it would send.

## Why it is not needed

Reading the server settled it. `campaigns.start` is already in the **`act`**
tier, and act-tier jobs enter at `awaiting_approval` and only become claimable
once a human POSTs to `/agent/jobs/:id/approve` from the extension. The MCP
surface deliberately exposes no approval tool, so a model cannot authorise its
own start.

That is stronger than the digest, and for the reason the digest spec had to
admit about itself: *"an assistant can call preview and confirm back to back
without surfacing either."* The act gate has no such weakness, because the
decision happens outside the model's reach entirely.

Adding the handshake on top would have put a second, weaker mechanism in front
of a working one, and taught readers that the approval rule lives in the
protocol rather than where it actually lives.

## What was actually broken

The gate existed; what the human saw through it did not.

`AgentApprovalsService.describe()` had cases for `send_message`,
`send_connection`, `like_post` and `comment_on_post`, each rendering the exact
text into a blockquote commented *"the exact text that would be sent. Shown in
full, never truncated."* `campaigns.start` fell through to the default branch,
so the approver saw the raw tool name, `{ "campaignId": "drip_…" }`, and a
dialog reading *"Linkedroid will campaigns.start. This happens on LinkedIn
straight away."*

Every message, the audience, even the campaign's name: none of it shown. The
one act-tier call that sends the most was the one approved blind.

**Fixed** by adding a `campaigns.start` case that resolves the campaign and
renders its name, audience and every message in order, and by `campaigns_get`
supplying the same content to the model. See
[campaigns-get-spec.md](campaigns-get-spec.md).

## The general lesson

The digest was designed against the tool list, without reading how the system
already enforced things. It proposed rebuilding, in the layer the model can
see, a guarantee that was already implemented in the layer it cannot — and
would have left the real defect, an approval screen that showed an opaque id,
completely untouched.

Read the enforcement before designing the safeguard.
