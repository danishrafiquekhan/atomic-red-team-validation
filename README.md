# atomic-red-team-validation

Writing a detection rule is one thing. Knowing it actually fires on the behaviour it's supposed to catch is a different thing entirely, and I think it's the part a lot of portfolios skip. This is where I run real Atomic Red Team tests in an isolated VM and check whether the rules in `detection-engineering` actually catch what I claim they catch.

Each case study follows the same shape: what I ran, what telemetry it produced, which rule was supposed to catch it, and whether it actually did. If a rule misses, that goes in the write-up too — a missed detection with an honest explanation is more useful to show than a rule I never actually tested.

Nothing here has been run yet — see `LAB-SETUP.md` for why (short version: it needs a Windows VM I haven't built yet, on purpose, since it's a multi-GB download with a EULA I want to actually read first, not something to grab automatically).

Everything runs in an isolated VM, snapshotted before each session, never on my actual network. No real device names, usernames, or IPs ever make it into `case-studies/*/evidence/`.

## One-time setup after cloning
```bash
git config core.hooksPath .githooks
```
