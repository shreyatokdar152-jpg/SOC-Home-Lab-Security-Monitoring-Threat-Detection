# Use Case: Suspicious PowerShell Execution Detection

## Objective

Detect suspicious PowerShell usage on a monitored Windows endpoint — specifically **encoded commands** (`-EncodedCommand` / `-enc`) and **execution-policy bypass** flags, both common techniques used by attackers and post-exploitation frameworks to evade logging and AV/EDR string-matching.

**MITRE ATT&CK mapping:** [T1059.001 – Command and Scripting Interpreter: PowerShell](https://attack.mitre.org/techniques/T1059/001/)

## Prerequisites

- Wazuh agent installed on the Windows endpoint
- **PowerShell Script Block Logging** enabled via Group Policy or registry, so PowerShell writes Event ID `4104` to the Windows Event Log, which the Wazuh agent then forwards:

```
Computer Configuration → Administrative Templates → Windows Components →
Windows PowerShell → Turn on PowerShell Script Block Logging → Enabled
```

Without this, PowerShell activity is invisible to the agent — Wazuh reads what the OS logs, it doesn't hook the PowerShell engine itself.

## How it works

Wazuh's default Windows event decoder parses PowerShell Script Block Logging events (Event ID 4104) from the `Microsoft-Windows-PowerShell/Operational` channel. This custom rule matches on that decoded event when the script block text contains indicators commonly associated with obfuscated or defense-evading PowerShell execution:

| Indicator | Why it matters |
|---|---|
| `-EncodedCommand` / `-enc` | Base64-encodes the actual command, hiding it from plain-text log review and simple keyword AV signatures |
| `-ExecutionPolicy Bypass` / `-ep bypass` | Explicitly disables the script-execution restriction, a strong signal of non-standard/automated tooling |
| `IEX` / `Invoke-Expression` combined with a download cmdlet (`DownloadString`, `Net.WebClient`) | Classic "fileless" download-and-execute pattern |

The rule uses a case-insensitive PCRE2 regex with word boundaries to avoid matching legitimate substrings inside unrelated commands.

## The rule

See [`rule.xml`](rule.xml).

## Testing / attack simulation

On a lab Windows endpoint with the Wazuh agent installed, the following was run in an elevated PowerShell prompt to simulate an obfuscated command similar to what post-exploitation tooling (e.g., Empire, Cobalt Strike PowerShell loaders) commonly generates:

```powershell
$cmd = "Write-Host 'simulated suspicious activity'"
$bytes = [System.Text.Encoding]::Unicode.GetBytes($cmd)
$encoded = [Convert]::ToBase64String($bytes)
powershell.exe -NoProfile -ExecutionPolicy Bypass -EncodedCommand $encoded
```

This produces a Script Block Logging event (Event ID 4104) containing both the `-ExecutionPolicy Bypass` and `-EncodedCommand` flags, which the Wazuh agent forwards to the manager.

### Validation with `wazuh-logtest`

The captured Windows event XML was replayed through `wazuh-logtest` on the manager to confirm the custom rule fired at the expected severity before deploying it against a live endpoint.

## Result

The alert appeared in **Wazuh Dashboard → Threat Hunting**, filtered on rule ID `100020`, showing:
- The full (still-encoded) command line
- The parent process (`powershell.exe`) and host
- MITRE ATT&CK technique tag (T1059.001)

See [`screenshots/`](screenshots/) for the alert as it appeared in the dashboard.

## Suggested follow-up (not implemented here)

- Add a companion rule to decode common Base64 `-EncodedCommand` payloads automatically for faster triage.
- Correlate with Sysmon Event ID 1 (process creation) if Sysmon is deployed, to capture the full parent/child process chain.
- Consider an [Active Response](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html) script to kill the process or isolate the host on high-confidence matches.
