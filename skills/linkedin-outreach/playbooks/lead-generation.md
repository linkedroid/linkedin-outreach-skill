# Lead generation

Cold prospects who have never heard of the user. The hardest motion, and the
one most often run badly — usually by contacting too many people with one
message.

## Shape

```
degree_filter
  ├─ 1st → send_message          (they already know you; skip the invite)
  └─ 2nd/3rd → send_connection   (note is the first thing they ever see)
         └─ delay 2 days
              └─ condition: connected?
                   ├─ yes → delay 1 day → send_message
                   └─ no  → end
```

Use the `warm_outreach` template for this; it is already laid out this way.
Build a custom flow only when the user needs something this shape cannot do.

## Sourcing

`linkedin.run_search` with the tightest filters the user can give. Then read
the returned headlines before tagging — search matches keywords, not intent,
and a "founder" filter returns plenty of people who founded something
irrelevant five years ago.

Tag in tiers when quality varies. Eighty well-chosen prospects beat a thousand
loose ones: the message can be specific, the reply rate carries the campaign,
and a poor list burns names that cannot be re-approached.

## Qualifying: score against anchors, not impressions

Do not tag on a feeling about a headline. Agree the anchors with the user
first, then apply them consistently and show the working.

| Signal | Strong (2) | Weak (1) | Out (0) |
|---|---|---|---|
| Role | decision-maker for this purchase | influences it | neither |
| Company | matches size and stage | adjacent | wrong segment |
| Timing | visible trigger — hiring, funding, launch, recent post | none visible | contra-indicated |

Tier A is a 2 on role plus at least one other 2. Tier B is everything else
worth contacting. A 0 on role or company is out, not Tier B.

**Record the evidence, not just the score.** For each prospect keep the line
that justified it — the headline phrase, the recent post, the job change. Two
reasons. The user can audit the list before anyone is contacted, and that same
line is what makes the message specific later. A prospect whose evidence you
cannot state is a prospect whose message will be generic.

**Propose, then let the user adjust.** Show the scored list with proposed tiers
and the evidence beside each. Expect the user to move some — they know their
market. Tag only what survives that pass.

## Volume

Start at 20–30 new prospects a day for an account with no outreach history, and
hold there for a week before raising it. The risk is not a published threshold —
it is the *shape* of the account. An account that has never sent an invitation
and suddenly sends 200 is an obvious outlier; the same account climbing 20 → 40
→ 80 over a month is not.

## Messages

The connection note carries the campaign. It is 300 characters, it is the only
thing a 2nd-degree prospect sees, and most of them decide from it alone.

Say who you are and why *them*, specifically. Do not pitch — the invite is
asking for a connection, not a meeting, and asking for the meeting here is what
makes people decline.

```
Hi {{firstName}}, [[ai:Why::one line on why this specific person, referencing
their work at {{company}}, no pitch, under 140 characters]] — would be good to
connect.
```

The follow-up after connecting is where the offer goes. Keep it to one ask, and
make the ask small: a question is answered far more often than a calendar link
is clicked.

## The send/hold gate

Before `campaigns.start`, put this in front of the user and wait for an
explicit yes. Not a summary — the actual content that will go out.

```
Audience    : <tag>, N prospects (Tier A: n, Tier B: n)
Daily cap   : N/day  →  finishes in ~N days
Sequence    : the full flow, step by step, with delays
Connection  : <the note, verbatim, with character count>
Follow-up   : <the message, verbatim>
Excludes    : excludeContacted on/off, excluded tags
```

Every personalised claim in those messages must trace to evidence actually
seen. If an AI block invents a detail about someone, it goes out under the
user's name to a stranger — that is the failure mode that costs an account its
reputation, not volume.

Hold, and say so plainly, if the user has not seen the messages, the evidence
behind a tier is thin, or a live campaign already targets this tag.
