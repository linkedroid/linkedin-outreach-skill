# Linkedroid Skills

Agent skills for the [Linkedroid](https://www.linkedroid.com) MCP server. They
teach an AI assistant how to run LinkedIn outreach end to end: work out who to
target, source and qualify them, build the campaign, write the messages, get
the user's approval, launch, and read the results.

## What this is, and what it is not

The Linkedroid MCP already describes its own tools well — what each one does,
what it needs, what it refuses. These skills deliberately do **not** restate
that. They carry the part a tool schema cannot hold:

- what to establish **before** building anything
- which tools to use **in what order**, and when to stop and ask
- how to tell a prospect worth contacting from one who merely matches a filter
- what must never happen without the user seeing it first

## Install

Skills are directories of markdown. Point your client at `skills/`:

**Claude Code** — clone into your project and the skills are discovered
automatically:

```bash
git clone https://github.com/<org>/linkedroid-skills .claude/skills/linkedroid
```

**Claude Desktop / claude.ai** — upload the `skills/linkedin-outreach`
directory as a skill.

You also need the Linkedroid MCP server connected. The skills call its tools by
name and do nothing useful without it.

## Skills

| Skill | Use when |
|---|---|
| `linkedin-outreach` | The user wants leads, pipeline, a GTM motion, a campaign, or wants to reach people who engaged with a post |

## Requirements

These skills assume MCP tools that do not all exist yet. See
[docs/mcp-gaps.md](docs/mcp-gaps.md) for the audit. Three have written specs:

| Spec | Why |
|---|---|
| [context store](docs/context-store-spec.md) | Without it the assistant re-asks the user about their own business every session |
| [`campaigns_get`](docs/campaigns-get-spec.md) | Nothing can read a campaign's messages back, so the approval rule below is unfollowable for any campaign the assistant did not just write |
| [`campaigns_start` digest](docs/campaigns-start-digest-spec.md) | Makes that approval rule enforced rather than advisory |
| [`linkedin_account_health`](docs/account-health-spec.md) | Both ceilings that stop a campaign — LinkedIn's invite credits and the Linkedroid plan limits — are invisible over MCP, so a campaign can be built that cannot run |

## Safety

These skills can cause messages to be sent under the user's name to people who
did not ask to hear from them. Two rules are not negotiable and are written
into the skills themselves:

1. **Nothing sends without the user seeing the exact content first.**
   `campaigns.create` makes a draft on purpose; creating and starting are
   separate decisions. This currently rests on the skill being read — see the
   [start digest spec](docs/campaigns-start-digest-spec.md) for moving it into
   the tool, where it can actually be enforced.
2. **Every personalised claim traces to evidence actually seen.** An invented
   detail about a real person goes out under the user's name.

## Versioning

A skill encodes how the tools behave, so it versions with the MCP server.
When a tool's behaviour changes, the skill is reviewed in the same release.
See [CHANGELOG.md](CHANGELOG.md).

## Contributing

Before changing a skill, read
[the authoring guide](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).
The two rules that matter most here: keep `SKILL.md` under 500 lines, and only
add context the model does not already have.
