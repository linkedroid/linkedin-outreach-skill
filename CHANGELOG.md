# Changelog

## Unreleased

### Cut the skill from 645 lines to 107

Baseline evaluations against the live MCP with no skill loaded produced GTM
plans as good as the playbook prescribed, and in one respect better: an
unassisted run staggered its cold lane to start five days before launch, so
invitations convert to connections in time for the announcement. No playbook
said to do that.

So the strategy prose was documentation for a problem that does not exist, and
it is gone — five playbooks and three reference files, replaced by one
`SKILL.md`. What survived is the part the baselines got wrong or could not
know: that `run_search` re-targets a running campaign, that an AI block without
a `Label::` is silently dropped, that connection notes truncate at 300
characters, that messaging a 2nd-degree prospect fails rather than queues, and
that the Linkedroid plan caps campaign size independently of LinkedIn.

One caveat on the evidence: the baseline agents were told not to call write
tools, which probably primed the caution they showed. That contaminates exactly
the measurement the send/hold gate was meant to test, so the gate stays.

### Specs

- `campaigns_get` — read a campaign's messages back. Without it the approval
  rule is unfollowable for any campaign the assistant did not just write.
- `campaigns_start` digest confirmation — binds approval to specific content,
  so the rule is enforced rather than requested.
- `linkedin_account_health` — both ceilings that stop a campaign.
- The MCP gap audit now runs to ten entries plus two design changes.

## Earlier

- `linkedin-outreach`: first version. Intake, motion routing, the six-step
  build loop, a qualification rubric with evidence, and the send/hold gate.
- Documented the MCP capability gaps this depends on, with a spec for the
  context store.
