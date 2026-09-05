**T1059.001 — Command and Scripting Interpreter: PowerShell**

Run for real. Picked this one to sit next to T1078.004 on purpose — that case study is identity/sign-in abuse (the cloud control plane never sees a process execute), this one is host-level execution (a PowerShell process actually spawns, encodes, and runs something). Different kill-chain stage, different telemetry source, so the two case studies together actually say something about coverage instead of testing the same detection twice.

**What I ran**

Not Atomic Test #1 as originally planned — that turned out to be "Mimikatz" (downloads and runs a real credential dumper from a public PowerSploit URL), found via `-ShowDetails` (a dry-run preview) before ever executing anything. Checking the technique's full test list instead, Test #15 — "ATHPowerShellCommandLineParameter -EncodedCommand parameter variations" — is the actual benign encoded-PowerShell test I meant: no elevation, no malware download, just spawns `powershell.exe -EncodedCommand <base64>` with a harmless payload.

`Invoke-AtomicTest T1059.001 -TestNumbers 15` itself hung indefinitely when run over PowerShell remoting (WinRM) — a `-TimeoutSeconds` value up to 120s made no difference, it hit the timeout every time. The underlying harness command it wraps (`Out-ATHPowerShellCommandLineParameter`, from the `AtomicTestHarnesses` module) ran fine when called directly instead of through `Invoke-AtomicTest`'s process-management layer — a WinRM-specific quirk in the wrapper, not a problem with the technique itself:
```powershell
Import-Module AtomicTestHarnesses -Force
Out-ATHPowerShellCommandLineParameter -CommandLineSwitchType Hyphen -EncodedCommandParamVariation E -Execute -ErrorAction Stop
```
This spawned a real `powershell.exe -NoProfile -E <base64>` process (base64-decodes to a harmless `Write-Host <test GUID>`), confirmed by the harness's own `TestSuccess: True` result and a real process ID.

**What it actually produced**

Real Sysmon Event ID 1 (process creation), captured with full context: command line (`powershell.exe -NoProfile -E VwByAGkAdABlAC0ASABvAHMAdAAgADYAMAAxADIANgAxADYAOAAtAGEANgBhAGIALQA0ADYAMQA1AC0AOQA0AGYAMAAtADAAMwA0ADIANAAxADMANgBiAGEAMgBjAA==`), file hashes (MD5/SHA256/IMPHASH of `powershell.exe` itself), integrity level (High), and — a genuinely useful detail I hadn't expected — the parent process wasn't a normal shell, it was `wmiprvse.exe` (WMI Provider Host), meaning `AtomicTestHarnesses` launches its target process via WMI rather than a direct child-process spawn. That's a real, different execution pattern from what I'd assumed going in, and it changed which MITRE technique the alert correctly mapped to (see below).

A side effect worth documenting honestly: the same PowerShell invocation also triggered 36+ separate Sysmon Event ID 11 (file-create) events for `.dll`/`.cmdline` files under `AppData\Local\Temp`, created by `csc.exe` (the C# compiler) and `powershell.exe` itself — this is PowerShell's own script-block-to-assembly compilation caching temp files, a completely normal, non-malicious side effect of any PowerShell execution on this Windows version, not an artifact specific to the encoded-command technique.

**Which rule should catch this**

Not a custom rule from `detection-engineering` — Wazuh's own **built-in** Sysmon-aware ruleset caught it, specifically rule `92071`: *"A powershell process created by WMI executed a base64 encoded command"*, tagged with both **T1047** (Windows Management Instrumentation) and **T1059.001** (PowerShell) — the WMI-launch detail from above is exactly why both techniques are tagged, not just the one I set out to test.

**Did it catch it**

Caught clean, first try, no tuning needed — level 12 alert, correct dual MITRE mapping, full forensic context (command line, hashes, parent process chain) captured automatically by Wazuh's default ruleset. The real gap wasn't detection logic at all: Wazuh's Windows agent doesn't monitor the Sysmon event channel (`Microsoft-Windows-Sysmon/Operational`) by default — only Application/Security/System are configured out of the box. Without adding that channel explicitly to the agent's `ossec.conf`, **none** of this would have ever reached Wazuh, regardless of how good the detection rule itself is. That's the actual lesson from this case study: the rule was never the weak link, the telemetry pipeline getting the right data to it was.

Also worth noting honestly: the 36+ file-create alerts from normal PowerShell compilation caching are a real, live example of the alert-fatigue problem discussed elsewhere in this portfolio — a legitimate, everyday PowerShell execution path produces enough Sysmon noise on its own to bury a single meaningful detection in volume, if nothing downstream tunes for it.
