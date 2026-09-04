**atomic-red-team-validation**

Writing a detection rule is one thing. Knowing it actually fires on the behaviour it is supposed to catch is a different thing entirely, and I think it is the part a lot of portfolios skip. This is where I run real Atomic Red Team tests in an isolated VM and check whether the rules in `detection-engineering` actually catch what I claim they catch.

Each case study follows the same shape: what I ran, what telemetry it produced, which rule was supposed to catch it, and whether it actually did. If a rule misses, that goes in the write-up too. A missed detection with an honest explanation is more useful to show than a rule I never actually tested.

Nothing here has been run yet. The isolated Windows VM these tests need now exists (built via QEMU, Windows Server 2022 evaluation, unattended install) — see `LAB-SETUP.md`. It doesn't have Sysmon or a Wazuh agent installed on it yet, so it can't generate the telemetry these case studies need. That's the actual next step, not this repo's current state.

Everything runs in an isolated VM, snapshotted before each session, never on my actual network. No real device names, usernames, or IPs ever make it into `case-studies/*/evidence/`.

**One-time setup after cloning**
```bash
git config core.hooksPath .githooks
```

For how the VM was actually built (unattended install, the real bug hit in the process, why BIOS/IDE over UEFI/virtio) see [Part 3.7–3.9](https://github.com/danishrafiquekhan/security-lab-notes/blob/main/parts/03c-tools-qemu-terraform-detection-as-code.md) and [Part 10](https://github.com/danishrafiquekhan/security-lab-notes/blob/main/parts/10-troubleshooting-log.md) of `security-lab-notes`.
