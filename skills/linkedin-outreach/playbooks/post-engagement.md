# Post engagement

Reaching people who reacted to or commented on a post. The warmest cold
audience there is: they have already shown interest in a topic, and you can say
why you are writing without inventing a reason.

## Why this converts better

Every other motion opens with "I found you in a search". This one opens with
something true and specific: they engaged with a post about X. Use that, and
the first line writes itself.

## Sourcing

`linkedin.get_post_engagers` on the relevant post. Use the user's own post when
there is one — reactions to their content are the strongest signal available.
A competitor's or an industry post also works, and is often a larger pool.

`linkedin.get_last_post` finds the user's most recent post if they do not have
a URL to hand.

Commenters are worth more than reactors. A comment costs effort; a reaction
costs a tap.

## Shape

```
degree_filter
  ├─ 1st → send_message
  └─ 2nd/3rd → send_connection (note references the post)
        └─ delay 2 days
             └─ condition: connected?
                  ├─ yes → delay 1 day → send_message
                  └─ no  → end
```

The `content_engagement` template is built for this.

## Messages

Name the post. Not "I saw you're interested in AI" — say which post, and what
they said if they commented.

```
Hi {{firstName}}, you commented on [the post about X] — [[ai:Hook::one line
responding to the specific point they made, as a peer not a seller, under 120
characters]]
```

If an AI block cannot see what they actually said, it will invent something
plausible. Pass the comment text or leave the hook generic; a vague true line
beats a specific false one.

## Timing

Run this within a few days of the post. The reference stops being warm quickly,
and "you commented on my post three months ago" reads worse than not mentioning
it at all.

## Send/hold gate

Same as lead generation: show the audience, the sequence and the verbatim
messages, and wait for a yes. See
[lead-generation.md](lead-generation.md#the-sendhold-gate).
