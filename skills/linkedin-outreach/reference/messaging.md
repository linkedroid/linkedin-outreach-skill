# Message craft

## Contents
- The syntax
- The `Name::` trap
- Length limits
- What earns a reply
- Evidence and invention

## The syntax

Message content is one string mixing three things:

| Form | Example | Notes |
|---|---|---|
| Literal text | `Hi there` | As written |
| Variable | `{{firstName}}` | `firstName`, `lastName`, `company`, `location` |
| AI prompt | `[[ai:Hook::write one line about…]]` | Generated per prospect |

Set with `campaigns.set_message`, never in the flow node.

## The `Name::` trap

An AI block **must** carry a label before `::`:

```
[[ai:Hook::one line on why this person, under 120 characters]]   ✅
[[ai:one line on why this person, under 120 characters]]         ❌ silently discarded
```

Without the label the block is dropped and the message sends with a gap where
the personal part should be. Nothing errors. Check every block has a label
before starting a campaign.

## Length limits

- **Connection note: 300 characters**, hard. LinkedIn truncates silently past
  it. An AI block that runs long costs the end of the sentence — always give
  blocks an explicit character budget, and leave room for the literal text
  around them.
- Messages have no hard cap, but a first message over about 120 words reads as
  a pitch regardless of content.

## What earns a reply

**One ask.** Two questions in one message get zero answers more often than one.

**A question beats a link.** "Is this something you deal with?" is answered far
more often than a calendar URL is clicked, and an answer starts the
conversation the link was trying to skip.

**Specific beats warm.** "Loved your post" is not personalisation. "Your point
about X" is. If there is nothing specific to say, the prospect probably should
not have been tagged.

**Say why them.** A 2nd-degree prospect's first question is "why am I getting
this". Answer it in the first line or they stop reading.

## Evidence and invention

Every personalised claim must trace to something actually seen — the headline,
a post, a job change. An AI block asked to write about someone it has no
information on will produce something plausible and wrong, and that goes out
under the user's name to a stranger.

When there is no evidence, write a good generic line instead. A true general
message outperforms a false specific one, and cannot embarrass the user.
