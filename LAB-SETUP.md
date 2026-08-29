# Lab setup — Atomic Red Team (safety-first)

This is the only repo in the portfolio that involves actually executing attack-simulation code, so it gets its own hard safety rules. Follow all of them, in order, before running a single Atomic test.

## Why this matters
Atomic Red Team tests intentionally reproduce real attacker behaviour (credential dumping, persistence, discovery commands, etc). Run on the wrong machine or the wrong network, that's not a simulation anymore — it's a real incident on your own device or home network.

## Hard rules
1. **Never run Atomic Red Team on this Mac (the host) directly.** Everything runs inside an isolated VM.
2. **VM networking must be NAT-only or Host-only — never Bridged.** Bridged mode puts the VM on your real home network, which defeats the isolation. UTM (already installed) defaults new VMs to "Shared Network" (NAT) — leave it there and do not switch to "Bridged."
3. **Disable shared clipboard and shared folders** between the VM and this Mac while tests run — a test that manipulates the clipboard or drops a file could otherwise cross the boundary.
4. **Snapshot the VM before every test session**, and roll back after — don't let a compromised-by-design VM accumulate state across sessions.
5. **Never reuse real credentials, work accounts, or your real email/identity inside the VM.** Use a throwaway local Windows account.
6. **Keep the VM off entirely when not actively testing** — don't leave it running as a standing target.

## One-time setup (UTM — already installed on this Mac)
1. Download a legitimate Windows evaluation VM image yourself from Microsoft (e.g. the [Windows 11 Enterprise evaluation](https://www.microsoft.com/evalcenter/evaluate-windows-11-enterprise)) — this is a deliberate manual step: it's a multi-GB download and you should read Microsoft's own EULA before agreeing to it, not have it fetched silently on your behalf.
2. In UTM: **New VM → Virtualize → Windows**, point it at the downloaded ISO.
3. Network: leave it on the UTM default (**Shared Network / NAT**). Do not change to Bridged.
4. Sharing: leave **Shared Directory** and clipboard sharing off.
5. After Windows installs, inside the VM: install PowerShell 7 and the [Atomic Red Team](https://github.com/redcanaryco/invoke-atomicredteam) PowerShell module, following their install instructions **inside the VM only**.
6. Take a UTM snapshot of this clean state before running any test. Restore to it after each session.

## Running a test (from inside the VM, not this Mac)
```powershell
# Example only — inside the isolated VM, never on the host
Install-Module -Name invoke-atomicredteam -Scope CurrentUser
Import-Module invoke-atomicredteam
Invoke-AtomicTest T1078.004 -ShowDetailsBrief
```

## Getting evidence back out safely
Don't use a live shared folder. Instead:
- Take a screenshot inside the VM, review it for anything sensitive, then transfer just that file out via UTM's one-off file copy (or a throwaway USB image), not a persistent shared mount.
- Sanitise device names, usernames, and IPs before it ever lands in `case-studies/*/evidence/` in this repo — see the root README's security note.

## After each session
- Roll back the VM to the clean snapshot.
- Power the VM off.
