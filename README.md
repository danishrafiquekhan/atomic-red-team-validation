# Atomic Red Team Validation

**Status: in progress** — attack simulation write-ups from isolated lab testing, used to validate the detections in [detection-engineering](https://github.com/danishrafiquekhan/detection-engineering).

## What this is
Case studies that run an Atomic Red Team test against a specific MITRE ATT&CK technique, capture the resulting telemetry, and check whether a detection actually fires on it.

## Why I built it
Writing a detection rule and *validating* it are different skills. This is where I prove rules actually catch the behaviour they claim to.

## How it works
Each case study in `case-studies/` follows: attack → telemetry generated → detection → outcome, with sanitised evidence (screenshots/log excerpts).

## What I learned / trade-offs
_(filled in per case study — false positives, blind spots, and detection gaps found go here)_

## Lab safety
All tests are run in an isolated VM (snapshotted beforehand), never against a production or home network. No real device names, usernames, or IP ranges are committed — see `case-studies/*/evidence/`.

## One-time setup after cloning
```bash
git config core.hooksPath .githooks   # enables the gitleaks secret-scan on commit
```
