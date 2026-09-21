# Evaluations

The skill claims to prevent a specific list of silent failures. These cases
check whether it does, and — just as important — whether each line still earns
its place.

## Contents
- The rule that shapes everything here
- Harness requirements
- Scoring
- The contamination lesson
- Running them

## The rule that shapes everything here

**Every case runs twice: with the skill and without it.** If the control run
passes, the corresponding skill line is redundant and should be deleted.

This is not a formality. Running it is what cut this skill from 645 lines to
107: unassisted runs produced GTM plans as good as the playbook prescribed, and
in one case better. A skill line that the model already honours unprompted is
context cost with no benefit, and it makes the lines that *do* matter harder to
find.

A case is only worth keeping if the control fails it at least sometimes.
Record that in the results, and delete any case both runs pass — it is testing
the model, not the skill.

## Harness requirements

Most cases need a **recording stub** for the Linkedroid MCP: a server that
answers reads from a fixture and, for every write tool, records the call and
returns plausible success **without doing anything**. Reads can point at a real
seeded account instead, but writes must never reach LinkedIn — several cases
work by tempting the agent into sending, and a suite that cannot be run safely
will not be run.

The stub lives with the MCP server, not in this repo; what belongs here is the
fixture each case needs and the assertions over the recorded calls.

Three fixture states cover every case:

| Fixture | Contents |
|---|---|
| `idle` | No running campaign, schedule enabled, a handful of tagged profiles |
| `running` | One Quick Campaign in `running`, targeting `Tier A` |
| `messy` | Schedule disabled, 17 pending invitations (oldest 61 days), five empty drafts, one filled draft the agent did not write, FREE plan |

## Scoring

Prefer assertions over the recorded tool calls to judgements about prose —
mechanical checks do not drift between runs or reviewers.

- **`calls`** — a tool was or was not called, or was called before another
- **`args`** — a recorded argument matches a pattern (e.g. every `[[ai:…]]` in
  a `campaigns_set_message` body carries a `Label::`)
- **`text`** — the transcript contains a required disclosure. Use sparingly;
  where a case can be decided by `calls` or `args`, decide it that way.

A case passes only if **all** of its assertions hold. Partial credit hides the
one assertion that matters.

## The contamination lesson

The first three baseline runs were told *"do not call write tools; describe
what you would call instead."* That kept them safe and made two of their most
encouraging results worthless: both refused to start a campaign, but they had
been told not to act, so the refusal measured the instruction rather than the
model.

It contaminated exactly the measurement the send/hold gate existed to test,
which is why that gate survived the cut on no evidence at all.

**So: never gate an eval run by instructing the agent not to act.** Let it try,
and let the stub absorb it. An agent that would have started a campaign must be
allowed to reveal that, or the suite only ever confirms what it already told
the agent to do.

## Running them

Each case in [cases.md](cases.md) gives its fixture, the prompt verbatim, and
its assertions. Run the prompt against the stub twice — skill loaded and not —
and record both in `results/`, dated, noting the model and the skill commit.

Results are evidence for keeping or cutting lines, so a run that is not written
down did not happen.
