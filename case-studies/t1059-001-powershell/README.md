**T1059.001 — Command and Scripting Interpreter: PowerShell**

Not run yet — waiting on the VM (see the root `LAB-SETUP.md`). Filling this in as a template so I know exactly what to capture once I do run it.

Picked this one to sit next to T1078.004 on purpose — that case study is identity/sign-in abuse (the cloud control plane never sees a process execute), this one is host-level execution (a PowerShell process actually spawns, encodes, and runs something). Different kill-chain stage, different telemetry source, so the two case studies together actually say something about coverage instead of testing the same detection twice.

**What I ran**
(which specific Atomic test, and the exact command)

Planned: Atomic Test #1 for T1059.001 ("PowerShell EncodedCommand"), run from inside the isolated VM per `LAB-SETUP.md`:

```powershell
Invoke-AtomicTest T1059.001 -TestNumbers 1
```

This runs `powershell.exe -enc <base64>` with a benign encoded command (writes a marker string to a local file), which is the same invocation shape as a real encoded-download-cradle or AMSI-bypass one-liner, just without the malicious payload.

**What it actually produced**
(the telemetry/logs it generated — not what I assumed it would produce, what it actually did)

**Which rule should catch this**
(pointing back at the matching rule in detection-engineering — probably something like powershell-encoded-command.yml or suspicious-powershell-execution.yml for this technique, keying off Sysmon Event ID 1 / Windows Event ID 4688 process creation with `-enc`/`-EncodedCommand` in the command line)

**Did it catch it**
(caught clean / caught but needed tuning / missed entirely — and why, if I know why)
