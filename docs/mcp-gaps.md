# MCP capability gaps

Audited against the Linkedroid MCP tool surface on 2026-09-16 by walking the
end-to-end journey a commercial skill has to complete.

**Seven of ten are now shipped** (2026-09-21), taking the surface from 31 tools
to 41: `campaigns.get`, `profiles.list_tagged`, `campaigns.delete`,
`linkedin.withdraw_invitation`, `campaigns.set_schedule`,
`linkedin.account_health`, and the four `context.*` tools.

Most of them turned out not to be missing capabilities at all. The extension
could already withdraw an invitation, delete a campaign and list who carries a
tag — the manifest simply never advertised any of it. Where real work was
needed it was usually one layer over from where this document pointed: the
withdraw tool was trivial, but `get_pending_invitations` projected the
invitation id away, so the id the write needed never reached the model.

The lesson for the rest of this list: check whether the capability exists
before designing it.

## Contents
- What is already covered
- The gaps, in priority order
- Design changes worth making
- A side effect worth guarding

## What is already covered

Sourcing, qualifying, tagging, campaign construction, launch, monitoring and
the inbox are all served. Two tools are notably better than typical:

- `campaigns_status` returns `blockedBy` (`COMPLETED`, `NO_PROSPECTS`,
  `PAUSED`, `OUTSIDE_SCHEDULE`) and `needsContent`. A skill can explain why
  nothing is sending instead of guessing, and can avoid "restarting" a campaign
  that is merely outside its schedule.
- `linkedin_list_conversations` warns that message text is untrusted input.
  That is the right call: replies are written by strangers and a skill that
  treats them as instructions is a prompt-injection hole.

## The gaps, in priority order

### 1. No persistent context store — SHIPPED

There is nowhere to keep the user's ICP, offer or writing voice, so every
session starts by re-interrogating them about their own business. That is the
experience the product exists to replace.

Everything else on this list is an improvement. This one is the difference
between a good prompt and a product.

Shipped as `context.set` / `get` / `list` / `delete` (tier `author`).
Spec: [context-store-spec.md](context-store-spec.md).

**One change from the spec.** It argued for server-side storage. As built it is
local, in the extension, beside the user's tags and campaigns — because every
route in this architecture executes in the extension anyway, and because
campaigns and tags are already device-local. Server-side storage would have
made context the only thing that synced, which is a more confusing product than
one that consistently does not.

The cost is real and should be said plainly: context does not follow a user to
a second machine. It is scoped per LinkedIn identity, so an agency switching
accounts gets that account's context, which is right.

`context.get` distinguishes a key never set from a key set to empty. That is
the difference between asking an onboarding question and not asking it again,
and it is the only reason `missing` exists as a separate field.

### 2. No way to read back a campaign's messages — SHIPPED

`campaigns_status` returns node content only for nodes that are **missing**
content (`needsContent`). For a campaign that is fully filled in it returns
nothing about the messages or the step sequence. `campaigns_list_node_types`
returns the generic catalogue, not this campaign's configured flow.

So an assistant cannot show the user what an existing campaign will actually
say before starting it. The skill's central safety rule — *nothing sends
without the user seeing the exact content first* — is only satisfiable for a
campaign the assistant wrote itself in the same session. For anything created
earlier, or by the web UI, or by a colleague, it cannot be honoured at all.

Found by running a baseline evaluation, not by reading the tool list: an
assistant with no skill loaded was asked why campaigns were not sending, and
reported it could not tell the user what either campaign would say.

Shipped as `campaigns.get` (tier `author`). It returns the full flow — nodes,
edges, and the content of every message, comment and connection note — with
per-message character counts and per-block AI validation. It also feeds the
approval card, which had the same blindness in a worse place.
Spec: [campaigns-get-spec.md](campaigns-get-spec.md).

### 3. No outcome metrics

`campaigns_stats` reports contacted, failed and remaining — activity, not
results. Missing: invitation acceptance rate, reply rate, and per-node or
per-variant performance.

Consequences: a retro cannot answer "which message worked", the `ab_split` node
cannot be scored so the feature is decorative, and commercially there is no way
to show a customer what their spend produced — which is what renewals turn on.

Suggested: extend `campaigns_stats` with `accepted`, `replied`, and a per-node
breakdown, rather than adding a tool.

### 4. `profiles_list_tags` returns counts, not members — SHIPPED

A tag's size is visible; who is in it is not. So an assistant cannot check
whether the people it is about to tag are already in another tag, or whether
two campaigns' audiences overlap.

That matters because contacting the same person from two campaigns is the most
damaging thing outreach does to a reputation, and `excludeContacted` only
covers people already *contacted* — not people queued in a second campaign that
has not started yet.

Found in a baseline run: an assistant tagging eleven new leads noted it could
not rule out double-tagging against the existing `Q4 Pipeline` tag.

Shipped as `profiles.list_tagged` (tier `read`). Returns who carries a tag,
with **every other tag each of them holds**, so an overlap between two
audiences is visible without a second call. `total` is how many carry the tag,
not the page size — a caller sizing an audience needs the former.

The membership was in the same stored record all along; only the count was
being exposed.

### 5. No facet resolution

`run_search` takes LinkedIn numeric ids for location, industry and company, and
its own description says to omit them unless you have real ids. So an ICP of
"seed-stage SaaS founders in the UK" degrades to keyword matching, and
geography — usually the second most important filter after title — is lost.

Suggested: `linkedin_resolve_facet(kind: 'geo'|'industry'|'company', query:
string)` returning candidate ids with names to choose from.

### 6. No withdraw invitation — SHIPPED

`get_pending_invitations` exists and its description notes stale invites are
"worth withdrawing" — but nothing can withdraw one. The skill can diagnose and
then do nothing.

This matters because a large pile of ignored invitations is one of the clearer
negative signals on an account.

Shipped as `linkedin.withdraw_invitation` (tier `act`, so a human approves each
one). It takes an `invitationId`.

`Linkedroid.withdrawPendingInvitation` already existed in `data_scrapper.js`
and was already routed, so the tool itself was a manifest entry. The actual
work was one layer over: `get_pending_invitations` runs through `pickProfile`,
a whitelist of profile fields, which dropped the invitation id — so every
invitation could be listed and none could be withdrawn. The adapter now carries
`invitationId` and `sentAt`, and nothing else from the raw record.

A caveat kept out of the tool description because it is ours, not LinkedIn's:
`withdrawPendingInvitation` uses `$.ajax` and treats any 200 as success, but
LinkedIn returns 200 on errors. A withdraw that fails server-side will report
success. Worth fixing when that code is next touched.

### 7. No account health — and the plan limits are invisible — SHIPPED

The skills advise a volume ramp while blind to the numbers that would inform
it. Two separate ceilings decide whether a campaign runs, and the MCP exposes
neither.

**LinkedIn's.** Invitation credits and any platform restriction. The extension
already reads the credit figure — `getPersonalInviteLimit()` in
`content_script/common.js` pulls `remainingCustomInviteCredits` straight from
LinkedIn — so this is plumbing an existing number through, not new work.

**Linkedroid's plan.** `PlanLimits` caps profile visits, scans, messages and AI
generations per day, and hard-caps campaign size, active campaigns, tags and
schedules. A FREE account allows 20 visits a day and 50 profiles per campaign.
Nothing over MCP says so, so an assistant can build a 400-prospect campaign for
that user, start it, and watch it silently stall. The user reads that as the
product not working.

That second half is the more commercially important one and it was missed in
the original audit — it is not a LinkedIn constraint at all, it is Linkedroid's
own, enforced in the Angular layer where the MCP cannot see it.

Suggested: `linkedin_account_health()` returning both ceilings.
Spec: [account-health-spec.md](account-health-spec.md).

Shipped as `linkedin.account_health` (tier `read`) — **one call**, both
ceilings, which needed four changes because the data lives on opposite sides of
the architecture:

1. **A relay.** Local tools can now call the content script
   (`callContentScript`). Routes are one kind or the other, so without this the
   tool would have had to be split in two and the caller made to correlate the
   halves — which defeats the point, since the question is "can this campaign
   run", not "here are two numbers".
2. **`Linkedroid.getInviteCredits`**, exposing `getPersonalInviteLimit()`
   through the message router for the first time. It asks LinkedIn each time
   rather than reusing the module-level cache the extension keeps, which
   drifts across a session and knows nothing about invitations sent from the
   LinkedIn UI in another tab.
3. **`PlanLimitService.publishPlanSnapshot()`**, writing the plan and its
   ceilings where offscreen can read them. Daily *usage* was already reachable
   under `planUsage_YYYY-MM-DD`; only the caps were missing, so an agent could
   see twelve visits used and not that the plan allowed twenty.
4. **The tool**, combining them with explicit `known: false` on either half.

**The `0` ambiguity is fixed at the boundary.** `getPersonalInviteLimit()`
returns `0` for a parse failure, a network failure and a genuinely exhausted
allowance alike. `getInviteCredits` maps everything that is not a positive
number or `-1` to `ok: false`, and the tool reports `known: false` with a
reason. A credit figure that could not be read is never reported as zero — an
agent told "0 credits left" would tell the user to stop sending for no reason.

A failed LinkedIn read also does not take the plan half down with it. Half an
answer beats none, as long as the missing half says it is missing.

The spec also records three bugs in the existing invite-credit code, all of
which err toward sending too much: the value is cached in a module variable and
decremented locally rather than refetched, and every failure path returns `0`,
which is indistinguishable from genuinely out of credits.

### 8. `linkedin_list_connections` times out

Called with no arguments in a baseline run, it returned no data. The assistant
was building a launch plan whose warm lane depended on the connection list, and
had to fall back to seven profile viewers — an audience far too small for the
job.

This is a reliability bug rather than a missing capability, but it has the same
effect: the warm audience, which is the cheapest and highest-converting one
available, cannot be reached. Needs pagination or a default limit.

### 9. No tool to configure the send schedule — SHIPPED

`campaigns_status` exposes `schedule.enabled: false`, and a baseline run
correctly flagged that sends would fire at any hour — then had to tell the user
to fix it by hand, because nothing exposes the setting.

Off-hours automation is one of the more visible tells, so a skill that can spot
the problem and not fix it is doing half a job.

Shipped as `campaigns.set_schedule` (tier `author`). It writes the same
`userSettings` shape `sw.js` reads and re-derives the alarms, refuses a window
that would allow nothing (`endHour` at or before `startHour`), refuses the same
weekday twice rather than silently keeping one, and refuses to do nothing
quietly. The description notes that it changes every campaign rather than one.

### 10. No campaign delete or archive — SHIPPED

Drafts accumulate in `campaigns_list` with no way to remove them. Cosmetic
until a user has thirty, at which point choosing the right one becomes a real
source of error.

Shipped as `campaigns.delete` (tier `author`). It refuses with
`CAMPAIGN_RUNNING` while a campaign is active — stopping and deleting are
different decisions, and halting live outreach must never be a side effect of
tidying up. There is no archive and the description says so rather than
implying one.

## Design changes worth making

Distinct from the list above: these tools exist and work, but their shape
pushes safety into prose where it cannot be enforced.

### The approval gate was fine; the approval screen was not — FIXED

This entry originally said the approval gate was advisory, resting on a skill
file being read. **That was wrong, and it was wrong because it was written from
the tool list without reading the server.**

`campaigns.start` is tier `act`: the job enters `awaiting_approval` and becomes
claimable only after a human approves it in the extension, and the MCP exposes
no approval tool, so a model cannot authorise its own start. The enforcement
was already there, in the one layer the model cannot reach.

What was broken is what the human saw. `describe()` rendered the exact text for
every other act-tier tool and let `campaigns.start` fall through to a raw
`{ campaignId }` dump — so the call that sends the most was the only one
approved blind. Fixed by rendering the campaign's name, audience and every
message on the card, and by `campaigns.get` giving the model the same content.

The withdrawn digest proposal is kept at
[campaigns-start-digest-spec.md](campaigns-start-digest-spec.md) for the
lesson: read the enforcement before designing the safeguard.

### `reason` is plumbed end to end and carries nothing

`AgentJob` stores a `reason`, the approval card renders it under a quote icon,
and `agent/mcp-server.js` hardcodes `reason: 'Requested via remote MCP'` on
every call. No tool schema asks the model for one, so every card shows the same
dead string where the justification should be.

Adding `reason` to the act-tier schemas is likely the cheapest real safety win
available: the approver would see *why* beside *what*, in a field that already
renders.

### `run_search` is tier `read` but mutates

Its own description says it navigates the user's working tab **and** re-targets
an active Quick Campaign. `read` tier means no approval is required. A
read-tier tool with a write side effect is a tier misclassification, and it is
the cleanest argument for fixing tools rather than documenting them: the skill
was carrying a warning that the tier system exists to make unnecessary.

## A side effect worth guarding

`run_search` navigates the user's working LinkedIn tab **and** re-targets an
active Quick Campaign. So a skill that searches while a campaign is running can
silently change that campaign's audience.

The skills guard this by checking `campaigns_status` before searching, but a
read-only search — one that returns results without steering the tab — would
remove the hazard rather than document it.
