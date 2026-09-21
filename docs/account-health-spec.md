# `linkedin_account_health` — MCP tool spec

**Status: implemented** as `linkedin.account_health`, tier `read`. See gap 7 in
[mcp-gaps.md](mcp-gaps.md) for the four changes it needed. The three bugs
listed below are fixed at the tool boundary, not in the original code — the
extension still caches and decrements the credit internally.

Surface the two ceilings that decide whether a campaign will actually run, so
volume advice stops being a guess.

## Contents
- The problem
- Two independent ceilings
- The tool
- Response shape
- Where each number comes from
- Three bugs to fix in the same change
- What not to claim

## The problem

The skills advise a volume ramp while blind to every number that would inform
it. Worse, an assistant can build a campaign that cannot run: nothing in the
MCP surface exposes the plan limits the extension itself enforces, so a
400-prospect campaign for a FREE user — capped at `maxProfilesPerCampaign: 50`
and 20 profile visits a day — is constructible, startable, and silently stalls.

The user experiences that as the product not working.

## Two independent ceilings

Either one stops a campaign, and they are enforced in different places.

**LinkedIn's.** Invitation credits, and whatever restriction the platform has
applied to the account. Hitting this damages the user's account, which is the
principal risk of the whole product.

**Linkedroid's plan.** `PlanLimits` in
`chrome-extension/src/app/shared/services/user.service.ts` — per-day caps on
visits, scans, messages and AI generations, plus hard ceilings on campaign
size, active campaigns, tags and schedules. Hitting this is a billing event,
not a safety one, but it stops the campaign just as dead.

An assistant needs both. Today it has neither.

## The tool

```
linkedin_account_health() -> AccountHealth
```

Read-only.

## Response shape

```jsonc
{
  "linkedin": {
    "remainingCustomInviteCredits": 34,   // LinkedIn's own number; null if unreadable
    "creditsUnlimited": false,            // true when LinkedIn reports no cap
    "pendingInvitations": 17,
    "oldestPendingDays": 61,
    "restriction": null                   // or a platform-reported restriction
  },
  "plan": {
    "name": "STARTER",
    "active": true,                       // false when EXPIRED
    "usage": {                            // today, resets at local midnight
      "profileVisits":  { "used": 12, "limit": 75,  "remaining": 63 },
      "profileScans":   { "used": 40, "limit": 200, "remaining": 160 },
      "messagesSent":   { "used": 3,  "limit": 50,  "remaining": 47 },
      "aiGenerations":  { "used": 18, "limit": 300, "remaining": 282 }
    },
    "ceilings": {
      "maxProfilesPerCampaign": 200,
      "activeDripCampaigns":    { "used": 1, "limit": 3 },
      "messageCampaigns":       { "used": 0, "limit": 5 },
      "schedules":              { "used": 0, "limit": 2 }
    },
    "features": { "salesNavigator": true, "aiPersonalizedMessages": true }
  },
  "warnings": ["PENDING_INVITES_HIGH", "SCHEDULES_UNUSED"]
}
```

`limit: -1` means unlimited and `remaining` is then `null`, matching
`withinLimit()`, which already treats `-1` as always-within.

## Where each number comes from

Most of this is already read somewhere in the extension. The tool is mainly
plumbing, not new reverse-engineering.

| Field | Source | Status |
|---|---|---|
| `remainingCustomInviteCredits` | `getPersonalInviteLimit()`, `content_script/common.js:4338` — the `voyagerRelationshipsDashCustomInviteComposeView` GraphQL call, field `remainingCustomInviteCredits` | exists |
| `pendingInvitations`, `oldestPendingDays` | `linkedin_get_pending_invitations` | exists |
| `plan.*` | `PlanLimits` + `DailyUsage` in `plan-limit.service.ts` | exists, not exposed over MCP |
| `restriction` | — | not currently detected anywhere |

**`remainingCustomInviteCredits` is the credit for invitations carrying a
note**, which is not the same as the total invitation allowance. Name it
precisely in the tool description. An assistant that reads it as "invitations
remaining" will advise sending noteless invites as though they were free, and
they are not.

**`restriction` should be `null` until something genuinely detects one.** A
field that always reports "healthy" because nothing populates it is worse than
no field: it converts an unknown into a false reassurance, and the skill will
quote it back to the user.

## Three bugs to fix in the same change

All three live in the existing invite-limit code and all three make the number
wrong in the direction of sending too much.

**It is fetched once and then decremented locally.** `personalInviteLimit` is a
module-level variable (`common.js:4337`) initialised on first use and
decremented after each send (`common.js:398`, `common.js:666`). It never
refetches, so it drifts from LinkedIn's real figure across a long session and
knows nothing about invitations sent from the LinkedIn UI in another tab.

**It is module-level state in a context that gets torn down.** Per the repo's
own rule 8, this cannot be relied on to survive. It happens to persist in the
content script, but the value is a cache with no expiry and no invalidation.

**Every failure path returns `0`.** Parse failure, network failure, empty
elements — all return `0`, which is indistinguishable from *genuinely out of
credits*. The consumer cannot tell "LinkedIn says you have none left" from "we
could not ask". For the MCP tool, return `null` on failure and let the
assistant say it does not know; `0` must mean zero.

That last one is the important one. An assistant that reports "you have 0
invitation credits" when the call merely failed will tell the user to stop
sending, which is at least safe — but the same confusion the other way, once
someone adds a `credits > 0` guard, silently gates real work on a parse error.

## What not to claim

There is no published, reliable number for "how many invitations per week is
safe". The weekly figure widely cited is inferred from user reports, varies by
account age and acceptance rate, and LinkedIn does not document it.

So this tool reports what is measurable — credits, pending invitations,
acceptance rate, plan usage — and the skill's ramp advice stays explicitly
heuristic. Surfacing real numbers next to a guess makes the guess look
authoritative, which is the failure mode to avoid.
