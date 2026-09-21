# GTM launch

Taking a product, feature or market to a defined audience over a fixed window.
Different from lead generation in one way that changes everything: there is a
**date**, and a story that has to land in order.

## What makes this different

Lead generation asks "who might buy". A launch asks "who needs to hear about
this, in what order, by when". That means segments run as separate campaigns
with different messages, not one campaign with one message.

## Establish the launch first

On top of the four intake questions in SKILL.md:

- **What is launching, and what changes for the buyer?** Not the feature — the
  thing they can now do.
- **When?** Decides whether segments run in sequence or parallel.
- **Who already knows the user?** Existing connections hear first. They are
  1st-degree, cost nothing to reach, and their replies tell you whether the
  message works before it goes to strangers.

## Sequence the segments

Run them in this order, not simultaneously. Each one teaches you something the
next one uses.

| Wave | Audience | Template | Why first |
|---|---|---|---|
| 1 | 1st-degree who know the user | `message_blast` | Cheap, fast, honest feedback on the message |
| 2 | Warm — post engagers, profile viewers | `content_engagement` | Have shown interest; message refined by wave 1 |
| 3 | Cold — search by ICP | `warm_outreach` | Largest and most expensive; go last, with a proven message |

**Wait for wave 1 replies before launching wave 2.** That is the whole reason
to sequence. If wave 1 produces no replies, the message is wrong and sending it
to a cold audience wastes that audience permanently.

## Sourcing per wave

- Wave 1: `linkedin.list_connections`, filtered to the relevant segment
- Wave 2: `linkedin.get_post_engagers` on the launch post,
  `linkedin.get_profile_viewers`
- Wave 3: `linkedin.run_search` against the ICP

Tag each wave separately (`Launch W1`, `Launch W2`, `Launch W3`) and set
`excludeContacted: true` on every campaign. Overlap between waves is the most
likely failure here — a connection who is also a post engager will otherwise be
contacted twice.

## Messages

One idea per message. A launch tempts people into listing everything that is
new; the reply rate goes down with each addition.

Wave 1 can be direct, because there is an existing relationship:

```
Hi {{firstName}}, we just shipped [thing]. [[ai:Relevance::one line on why this
specific person would care given their role, under 140 characters]] — happy to
show you if useful.
```

Waves 2 and 3 need the connection note to earn attention before the launch can
be mentioned at all. Do not put the launch in the invite.

## Reading it

Check `campaigns_stats` per wave, not in aggregate. A strong wave 1 and a dead
wave 3 means the message depends on already knowing the user — which is worth
finding out, and invisible in a combined number.

## Send/hold gate

Per wave, separately. Approving a launch plan is not approving three campaigns'
worth of messages — show each wave's content before that wave starts.
