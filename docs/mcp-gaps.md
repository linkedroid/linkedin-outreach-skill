# MCP capability gaps

Audited against the Linkedroid MCP tool surface (~31 tools) on 2026-09-16 by
walking the end-to-end journey a commercial skill has to complete.

## Contents
- What is already covered
- The gaps, in priority order
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

### 1. No persistent context store — blocks the core promise

There is nowhere to keep the user's ICP, offer or writing voice, so every
session starts by re-interrogating them about their own business. That is the
experience the product exists to replace.

Everything else on this list is an improvement. This one is the difference
between a good prompt and a product.

Spec: [context-store-spec.md](context-store-spec.md).

### 2. No way to read back a campaign's messages — breaks the approval gate

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

Suggested: `campaigns_get(campaignId)` returning the full flow — nodes, edges,
and the content of every message, comment and connection note.

### 3. No outcome metrics

`campaigns_stats` reports contacted, failed and remaining — activity, not
results. Missing: invitation acceptance rate, reply rate, and per-node or
per-variant performance.

Consequences: a retro cannot answer "which message worked", the `ab_split` node
cannot be scored so the feature is decorative, and commercially there is no way
to show a customer what their spend produced — which is what renewals turn on.

Suggested: extend `campaigns_stats` with `accepted`, `replied`, and a per-node
breakdown, rather than adding a tool.

### 4. `profiles_list_tags` returns counts, not members

A tag's size is visible; who is in it is not. So an assistant cannot check
whether the people it is about to tag are already in another tag, or whether
two campaigns' audiences overlap.

That matters because contacting the same person from two campaigns is the most
damaging thing outreach does to a reputation, and `excludeContacted` only
covers people already *contacted* — not people queued in a second campaign that
has not started yet.

Found in a baseline run: an assistant tagging eleven new leads noted it could
not rule out double-tagging against the existing `Q4 Pipeline` tag.

Suggested: `profiles_list_tagged(tag, start?, limit?)` returning the public
identifiers carrying a tag.

### 5. No facet resolution

`run_search` takes LinkedIn numeric ids for location, industry and company, and
its own description says to omit them unless you have real ids. So an ICP of
"seed-stage SaaS founders in the UK" degrades to keyword matching, and
geography — usually the second most important filter after title — is lost.

Suggested: `linkedin_resolve_facet(kind: 'geo'|'industry'|'company', query:
string)` returning candidate ids with names to choose from.

### 6. No withdraw invitation

`get_pending_invitations` exists and its description notes stale invites are
"worth withdrawing" — but nothing can withdraw one. The skill can diagnose and
then do nothing.

This matters because a large pile of ignored invitations is one of the clearer
negative signals on an account.

Suggested: `linkedin_withdraw_invitation(publicIdentifier)`.

### 7. No account health

The skills advise a volume ramp while blind to the numbers that would inform
it: invitations sent today, acceptance rate over the last week, whether the
account is currently restricted.

For a product whose principal risk is the customer's account, that is the wrong
thing to be blind to. Advice given without the numbers is a guess presented as
a recommendation.

Suggested: `linkedin_account_health()` returning today's action counts, recent
acceptance rate, and any restriction the platform exposes.

### 8. No campaign delete or archive

Drafts accumulate in `campaigns_list` with no way to remove them. Cosmetic
until a user has thirty, at which point choosing the right one becomes a real
source of error.

## A side effect worth guarding

`run_search` navigates the user's working LinkedIn tab **and** re-targets an
active Quick Campaign. So a skill that searches while a campaign is running can
silently change that campaign's audience.

The skills guard this by checking `campaigns_status` before searching, but a
read-only search — one that returns results without steering the tab — would
remove the hazard rather than document it.
