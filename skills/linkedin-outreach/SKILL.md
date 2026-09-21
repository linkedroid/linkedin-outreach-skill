---
name: linkedin-outreach
description: Plans and runs LinkedIn outreach through Linkedroid — sourcing an audience, building a drip campaign, writing the messages, launching it and reading the results. Use when the user wants leads, pipeline, a go-to-market or GTM motion, prospecting, a LinkedIn campaign or drip sequence, wants to reach people who engaged with a post or viewed their profile, or wants to re-engage cold conversations.
---

# LinkedIn outreach

The Linkedroid tools document themselves. This is the part they cannot hold:
what to ask before building, which order to do things in, and when to stop and
check with the user.

## Settle four things first

Do not search, tag or create anything until these are answered. Ask the user;
do not assume. A campaign built on a guessed audience wastes the audience —
prospects can only be approached once.

1. **Who.** Job titles, seniority, industry, company size, geography. "Founders"
   is not an audience; "seed-stage B2B SaaS founders in the UK" is.
2. **Why them.** What the user is offering and why this group would care. This
   becomes the message; without it every message reads as a template.
3. **How many, and how fast.** A day's worth or a quarter's. Decides
   `audienceLimit` and whether to run one campaign or several.
4. **What counts as success.** A reply, a call booked, a follow. Decides the
   `goal` node and when to stop.

If the user cannot answer 2, stop and work on the offer before the campaign.
That is the most common reason outreach fails, and no sequence fixes it.

## Pick the motion

| The user wants | Read |
|---|---|
| New pipeline from cold prospects | [playbooks/lead-generation.md](playbooks/lead-generation.md) |
| To launch a product or enter a market | [playbooks/gtm-launch.md](playbooks/gtm-launch.md) |
| To reach people who engaged with a post | [playbooks/post-engagement.md](playbooks/post-engagement.md) |
| To revive old conversations or dormant contacts | [playbooks/re-engagement.md](playbooks/re-engagement.md) |
| To warm a network before selling anything | [playbooks/network-warming.md](playbooks/network-warming.md) |

Also available: [reference/flows.md](reference/flows.md) for flow shapes that
work, [reference/messaging.md](reference/messaging.md) for message craft, and
[reference/limits.md](reference/limits.md) for volume and safety.

## The build loop

Every motion follows the same six steps. The playbook changes what goes in
them, not their order.

```
- [ ] 1. Source the audience  (run_search / get_post_engagers / list_connections)
- [ ] 2. Qualify and tag      (profiles.tag)
- [ ] 3. Create the campaign  (campaigns.create, as a DRAFT)
- [ ] 4. Write the messages   (campaigns.status → campaigns.set_message)
- [ ] 5. Show the user        (full sequence, verbatim, before anything sends)
- [ ] 6. Start and watch      (campaigns.start → campaigns.stats)
```

**Step 2 is where judgement lives.** Search returns everyone matching the
filters; the tag should carry only the ones worth contacting. Read the
headlines. Tag tiers separately (`Tier A`, `Tier B`) when quality varies —
it lets the user send the strong message to the strong list.

**Step 5 is not optional.** Print the whole sequence — every message, every
delay, the audience size — and get an explicit yes. These messages go out under
the user's name to people who did not ask to hear from them. A campaign cannot
be recalled once it starts.

## Non-negotiables

**Never start a campaign the user has not seen.** `campaigns.create` makes a
draft on purpose. Creating and starting are separate decisions; keep them
separate.

**Match the template to connection degree.** `message_blast` and
`re_engagement` message before connecting, so they only work on 1st-degree
contacts. For 2nd or 3rd degree use a connect-first template — `warm_outreach`,
`connection_followup`, `sales_outreach`. Check with
`linkedin.get_member_distance` when unsure. Messaging a stranger fails; it does
not queue.

**Always set `excludeContacted: true`** unless the user says otherwise. Without
it two campaigns can hit the same person, which is the single most damaging
thing outreach does to a reputation.

**Put a delay between every pair of actions.** Back-to-back contact reads as
automation to the recipient, never mind the platform.

**Volume is the user's risk, not a number to maximise.** See
[reference/limits.md](reference/limits.md) before proposing daily caps.

## When something is already running

Check `campaigns.list` before creating anything. If a campaign targeting the
same tag is live, adding another one means contacting the same people twice.
Ask whether to extend the existing campaign instead.
