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
