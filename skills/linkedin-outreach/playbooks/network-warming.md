# Network warming

No messages at all. Profile visits and post engagement, so that when a real
approach comes later the name is familiar.

Use this when the user is new to outreach, when an account has no activity
history, or before a launch.

## Shape

```
profile_visit
  └─ delay 2 days
       └─ engage            (likes their most recent post)
            └─ delay 3 days
                 └─ tag "Warmed"
```

No `send_message`, no `send_connection`. That is the point — this motion builds
familiarity and account history without spending the one approach each prospect
allows.

## Why it is worth a campaign of its own

**It creates account history.** An account that has never sent an invitation
and suddenly sends 200 is an obvious outlier. Two weeks of visits first makes
the later ramp unremarkable. See [../reference/limits.md](../reference/limits.md).

**Profile visits are visible.** The prospect sees the user in "who viewed your
profile" — the cheapest impression available, and it costs nothing if they
ignore it.

**It costs nothing if it fails.** Nobody is contacted, so nobody is burned. The
audience stays fully available for a real campaign later.

## Volume

Visits tolerate far more than messages. 100 a day is reasonable from the start;
messages should not be.

`engage` does nothing when the prospect has not posted. That is a skip, not a
failure — do not report it as one.

## What comes next

Warming is a prelude. When it finishes, the `Warmed` tag is the audience for a
real campaign — and because those prospects have seen the user's name, the
connection note can reference it honestly.
