# Volume and safety

## Contents
- Why there is no safe number
- Suggested ramp
- What actually raises risk
- Degree rules

## Why there is no safe number

LinkedIn does not publish an automation threshold and the one it enforces is
not fixed — it varies with account age, connection count, history and reply
rate. Any specific number presented as "the limit" is a guess. What stays
consistent is the *shape* of the account, not the size of the number.

Tell the user this plainly rather than inventing a figure. They are the one
carrying the account risk, and a confident wrong number is worse than an honest
range.

## Suggested ramp

A starting point for an account with no automation history, not a guarantee:

| Week | Profile visits/day | Invitations/day | Messages/day |
|---|---|---|---|
| 1 | 100 | 0 | 0 |
| 2 | 100 | 20 | 20 |
| 3 | 150 | 40 | 40 |
| 4+ | read the numbers, then raise one step |

Change one thing at a time and hold it long enough to see the effect. Raising
three dials at once means a bad week tells you nothing about which caused it.

## What actually raises risk

- **A sudden jump.** Going from nothing to a high volume is the clearest signal.
- **Contacting the same person twice.** Set `excludeContacted: true`.
- **No delays.** Actions back-to-back on one prospect.
- **A loose audience.** A thousand wrong profiles produces low replies, and low
  reply rates attract attention on their own.
- **Invitations that get ignored.** A large pile of pending invitations is worse
  than a small pile of accepted ones. `linkedin.get_pending_invitations` lists
  them oldest first — but there is no tool to withdraw one, so report the stale
  ones and tell the user to withdraw them in LinkedIn by hand. Do not claim to
  have withdrawn anything.

## Degree rules

- **1st degree** — can be messaged directly. `message_blast` and `re_engagement`
  are for these people only.
- **2nd / 3rd degree** — must be connected to first. Use a connect-first
  template, or gate on a `degree_filter` node.
- Messaging a 2nd or 3rd-degree prospect **fails**; it does not queue and retry.
- When unsure, `linkedin.get_member_distance` answers it for one person.
