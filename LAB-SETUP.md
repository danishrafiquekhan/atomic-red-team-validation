**Lab setup — do this properly, not fast**

This is the one repo in the portfolio where I'm actually running attack code, so it gets stricter rules than everything else. Read all of this before running a single test, not after.

**Why I'm being this careful about it**
Atomic Red Team tests are real attacker behaviour — credential dumping, persistence mechanisms, discovery commands. Run one on the wrong machine or the wrong network and it's not a simulation anymore, it's just an incident, and it's mine to clean up. I'd rather be annoyingly careful here than have a "how I accidentally locked myself out of my own accounts" story to explain later.

**Rules I'm not skipping**
1. Never on this Mac directly. Everything happens inside an isolated VM.
2. VM networking stays NAT/Shared Network, never Bridged. Bridged puts the VM on my actual home network and defeats the whole point. UTM defaults to Shared Network already — I'm just not changing it.
3. Shared clipboard and shared folders off while a test is running. A test that messes with the clipboard or drops a file shouldn't have a path back to this Mac.
4. Snapshot before every session, roll back after. I don't want a VM that's accumulated three sessions' worth of "compromised on purpose" state.
5. No real accounts inside the VM — throwaway local Windows account only.
6. VM stays off when I'm not actively testing. Not leaving it running as a standing target for no reason.

**Getting the VM built**
1. Download a Windows evaluation image myself, directly from Microsoft (the Windows 11 Enterprise eval is the usual one). Not automating this — it's several GB and there's a EULA attached that I want to actually read, not click through blind.
2. UTM → New VM → Virtualize → Windows, point it at the ISO.
3. Leave networking on Shared Network (NAT). Don't touch Bridged.
4. Leave shared directories and clipboard sync off.
5. Once Windows is up: install PowerShell 7 and the Atomic Red Team module (`invoke-atomicredteam`) inside the VM, following their own install docs.
6. Snapshot that clean state before running anything. Roll back to it after.

**Running a test**
```powershell
# inside the VM only, never on the host
Install-Module -Name invoke-atomicredteam -Scope CurrentUser
Import-Module invoke-atomicredteam
Invoke-AtomicTest T1078.004 -ShowDetailsBrief
```

**Getting evidence out without breaking the isolation**
No live shared folder for this — screenshot inside the VM, check it for anything sensitive, then move just that one file out through UTM's one-off file copy. Sanitise hostnames/usernames/IPs before they land anywhere in this repo.

**After a session**
Roll the VM back to the clean snapshot, power it off.

**What actually happened, first real run — honest deviations from the plan above**

The first real technique execution (T1059.001, see `case-studies/t1059-001-powershell/`) didn't follow this document exactly, and it's worth saying plainly rather than pretending it did:

- **No snapshot was taken before the test.** The VM was built via raw QEMU (not UTM — documented elsewhere as a deliberate switch), and I never actually set up qcow2 snapshotting before running anything. The test itself was benign (a harmless encoded `Write-Host`), so nothing destructive happened, but the "snapshot before, roll back after" discipline this document calls for didn't happen this time. Real gap, not a hypothetical one — fixing this before running anything less benign than a Sysmon-detection-validation test is the honest next step.
- **Control happened over PowerShell remoting (WinRM) from the host, not from inside the VM's own console.** The commands that actually launched the attack technique were typed on the Mac and executed remotely, not typed at a keyboard sitting in front of the VM's display. The execution itself still happens entirely inside the isolated VM (the process spawns there, the telemetry comes from there) — but the control channel crossing from host to VM via WinRM is a different shape than this document originally assumed, and it's the reason WinRM's own port (5985) needed a host-to-VM port-forward that didn't exist in the original network plan.
- **The built-in Administrator account was used, not a separate throwaway local account.** Lower risk than it sounds given the VM has no real data or credentials on it and is fully isolated, but it's a real deviation from rule 5 above, not something to gloss over.

None of this changes the actual verified result (a real Sysmon-based detection genuinely fired, see the case study), but a document about doing this "properly, not fast" should say plainly where the first real run didn't fully live up to its own rules.
