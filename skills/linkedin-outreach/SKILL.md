---
name: linkedin-outreach
description: Plans and runs LinkedIn outreach through Linkedroid — sourcing an audience, building a drip campaign, writing the messages, launching it and reading the results. Use when the user wants leads, pipeline, a go-to-market or GTM motion, prospecting, a LinkedIn campaign or drip sequence, wants to reach people who engaged with a post or viewed their profile, or wants to re-engage cold conversations.
---

# LinkedIn outreach

The tools document themselves and you can plan the campaign. This carries only
what the schemas do not say: what fails silently, what cannot be undone, and
what a tool does beyond the obvious.

## Hazards

**`run_search` re-targets a running campaign.** It navigates the user's working
LinkedIn tab *and* repoints an active Quick Campaign at the new results. So
searching while a campaign runs can silently change who that campaign contacts.
Call `campaigns_status` first; if something is live, say so and ask before
searching.

**An AI block without a label is silently discarded.**

```
[[ai:Hook::one line on why this person, under 120 chars]]   ✅
[[ai:one line on why this person, under 120 chars]]         ❌ dropped, no error
```

The message then sends with a gap where the personal part should be. Check
every block for its `Label::` before starting.

**Connection notes truncate silently at 300 characters.** An AI block that runs
long costs the end of the sentence. Give every block an explicit character
budget and leave room for the literal text around it.

**Messaging a 2nd or 3rd-degree prospect fails — it does not queue.**
`message_blast` and `re_engagement` message before connecting, so they work on
1st-degree contacts only. For anyone else use a connect-first template
(`warm_outreach`, `connection_followup`, `sales_outreach`) or gate on a
`degree_filter` node. `linkedin_get_member_distance` settles one case.

**`excludeContacted: true` unless the user says otherwise.** Without it two
campaigns can hit the same person, which is the most damaging thing outreach
does to a reputation. `profiles_list_tags` returns counts, not members, so
overlap between two tags cannot be checked directly — ask the user rather than
assuming the lists are disjoint.

## Before starting anything

**Show the user every message, verbatim, and get an explicit yes.** These go
out under their name to people who did not ask to hear from them, and a
campaign cannot be recalled. `campaigns_create` makes a draft deliberately;
creating and starting are separate decisions.

For a campaign you did not write in this session, you cannot currently read its
messages back — `campaigns_status` returns content only for nodes that are
*missing* content. Say that plainly rather than starting something unseen, and
point the user at the campaign in the web UI.

**Check `schedule.enabled`.** When false, actions fire at whatever hour the
campaign starts, including 3am. No tool sets the schedule, so report it and ask
the user to fix it in the app.

**Check what is already running.** `campaigns_list` first: a second campaign on
the same tag means contacting the same people twice.

## Volume

There is no published safe threshold. What LinkedIn enforces varies with
account age, connection count, history and reply rate, so any specific figure
is a guess — tell the user that rather than inventing one. They carry the
account risk.

A starting ramp for an account with no automation history:

| Week | Visits/day | Invitations/day | Messages/day |
|---|---|---|---|
| 1 | 100 | 0 | 0 |
| 2 | 100 | 20 | 20 |
| 3 | 150 | 40 | 40 |
| 4+ | read the numbers, then raise one step |

Change one dial at a time. Three at once means a bad week tells you nothing.

What actually raises risk: a sudden jump from nothing; the same person
contacted twice; no delay between actions on one prospect; a loose audience,
because low reply rates draw attention by themselves; and a growing pile of
ignored invitations. `linkedin_get_pending_invitations` lists the stale ones
oldest first, but **nothing can withdraw them** — report them and ask the user
to do it by hand. Never claim to have withdrawn anything.

The Linkedroid plan is a second ceiling, separate from LinkedIn's: daily caps
on visits, scans, messages and AI generations, plus a hard cap on campaign
size. On a free account that is 20 visits a day and 50 profiles per campaign.
Nothing exposes these over MCP, so a campaign larger than the plan allows will
build, start, and quietly stall. When planning anything large, ask which plan
the user is on.

## Personalisation

Every specific claim must trace to something actually seen — the headline, a
post, a job change. An AI block asked to write about someone it knows nothing
about produces something plausible and wrong, under the user's name, to a
stranger. With no evidence, write a good generic line: a true general message
beats a false specific one and cannot embarrass anyone.

Ask for the ICP and the offer before building. Do not infer them from the
user's own headline — that produces a campaign aimed at a guess, and prospects
can only be approached once.
