**T1078.004 — Valid Accounts: Cloud Accounts**

Run for real, but adapted — not the official Atomic Red Team tests for this technique, and worth being upfront about exactly why.

**What I ran**

All three official Atomic Red Team tests for T1078.004 require real cloud resource creation: two use GCP (service account creation, custom IAM role), one uses Azure (`New-AzAutomationRunbook`, requiring `Connect-AzAccount` and a real `terraform apply` against a real Azure subscription to provision an Automation Account first). None of them have a benign local-only variant the way T1059.001's tests did — checked with `-ShowDetails` before deciding this, not assumed. Running the official Azure test would mean real cloud login and real resource creation, a different risk/cost category than everything else validated in this repo so far.

Instead, adapted the same underlying technique — a valid cloud identity establishing a persistence mechanism — using AWS IAM against LocalStack (free, local, already used elsewhere in this portfolio), since that's genuinely testable without touching a real cloud account:
```bash
aws --endpoint-url=http://localhost:4566 iam create-user --user-name backup-svc-account
aws --endpoint-url=http://localhost:4566 iam attach-user-policy --user-name backup-svc-account --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws --endpoint-url=http://localhost:4566 iam create-access-key --user-name backup-svc-account
```
The intentionally innocuous-sounding username (`backup-svc-account`) mirrors real attacker naming conventions for backdoor accounts meant to blend into a real user/role list. This is the actual intent behind T1078.004's persistence sub-technique: an adversary with initial access creates a *new* valid credential, so access survives even if the original compromised credential gets revoked — same intent as the official Azure Automation Runbook test, different cloud, no real resources touched.

**What it actually produced**

LocalStack Community doesn't generate CloudTrail-format logs at all (a real, already-documented limitation in `aws-identity-detection` — that's a Pro-only feature), so there's no CloudTrail-based detection surface here. What LocalStack *does* produce: its own request log, one line per API call, in a clean format — `AWS iam.CreateUser => 200`, `AWS iam.AttachUserPolicy => 200`, `AWS iam.CreateAccessKey => 200` — captured via `docker logs -f localstack-localstack-1`, filtered, and relayed into Wazuh the same on-demand pattern as this portfolio's other live sources.

**Which rule should catch this**

A new custom Wazuh rule set (Wazuh has no built-in AWS/CloudTrail decoder, same situation as Cloudflare and Auth0) matching on the specific IAM API operations that establish cloud-account persistence: `CreateUser`, `AttachUserPolicy`, `CreateAccessKey` (the last one tagged at a higher severity — level 9 vs level 7 — since minting a new credential is the actual persistence mechanism, the user/policy steps are precursors to it).

**Did it catch it**

Caught clean, all three steps, correct MITRE T1078.004 tagging on every alert:
```
rule 100032 (level 7): "Valid Accounts (Cloud): new IAM user created"
rule 100033 (level 7): "Valid Accounts (Cloud): IAM policy attached to a user"
rule 100031 (level 9): "Valid Accounts (Cloud): new IAM access key created — possible persistence via new credential"
```
This is genuinely a fifth live detection source in this lab (after Cloudflare, MySQL, Suricata, and Auth0), not a duplicate of anything else — LocalStack's raw request log is a source this portfolio hadn't wired into Wazuh before this case study. The relay ran on-demand for this test only, then was stopped — same discipline documented for the other live sources in `detection-engineering/local-lab/live-traffic-tests/README.md`.

**Being honest about what this doesn't prove**

This validates the *detection logic*, not the *official* Atomic Red Team test for T1078.004 — a reader checking this against the upstream Atomic Red Team library won't find a matching test, because the official ones need real cloud infrastructure this lab deliberately avoided touching. If the real Azure test ever gets run against an actual authenticated subscription, that would be a genuinely different, additional validation, not a replacement for this one.
