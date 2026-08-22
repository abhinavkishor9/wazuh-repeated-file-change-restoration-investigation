# Lab 58 — Troubleshooting Notes

## Issue 1 — Sysmon Configuration Required Administrator Privileges

### Command

```powershell
sysmon64.exe -c
```

### Error

```text
System Monitor v15.21 - System activity monitor
By Mark Russinovich and Thomas Garnier
Copyright (C) 2014-2026 Microsoft Corporation

You need to launch Sysmon as an Administrator.
```

### Cause

The Sysmon configuration command was initially executed from a non-elevated PowerShell session.

### Resolution

PowerShell was reopened using:

`Run as administrator`

The Sysmon configuration command could then be executed with administrative privileges.

### Lesson

Sysmon management and configuration commands may require an elevated PowerShell session.

---

## Issue 2 — Event ID 11 Query Returned No Target-File Events

### Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName="Microsoft-Windows-Sysmon/Operational"
    Id=11
} -MaxEvents 100 |
Where-Object {$_.Message -match "Important_Report.txt"} |
Select-Object TimeCreated, Id, Message
```

### Result

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

### Investigation

Initially, this suggested that Sysmon Event ID 11 might not be functioning.

However, later evidence showed that Event ID 11 was present.

A matching Event ID 11 event was found for:

`C:\Users\Dell\AppData\Local\Temp\__PSScriptPolicyTest_v42nuax3.b5n.ps1`

The event showed:

```text
Event ID:
11

Process:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

Process ID:
14700

UtcTime:
2026-08-22 02:09:17.810 UTC
```

### Conclusion

Sysmon Event ID 11 was operational.

The issue was specifically the lack of a matching Event ID 11 record for:

`C:\SOC-Lab\Important_Report.txt`

---

## Issue 3 — Event ID 23 Query Returned No Results

### Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName="Microsoft-Windows-Sysmon/Operational"
    Id=23
} -MaxEvents 100 |
Where-Object {$_.Message -match "Important_Report.txt"} |
Select-Object TimeCreated, Id, Message
```

### Result

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

### Explanation

The lab primarily used:

```powershell
Set-Content
```

The command modifies the contents of a file.

It does not explicitly delete the target file.

Therefore, an Event ID 23 result was not expected from the primary simulation.

### Lesson

Do not assume that every file modification produces a deletion event.

---

## Issue 4 — Script Content Needed Verification

The PowerShell simulation script was created using a here-string.

The script was then verified using:

```powershell
Get-Content "C:\SOC-Lab\FileChangeSimulation.ps1"
```

The displayed script contained repeated `Set-Content` operations.

The output showed:

```powershell
$file = "C:\SOC-Lab\Important_Report.txt"

Set-Content $file "Modified content - Script Change 1"
Start-Sleep -Seconds 3

Set-Content $file "Original report content"
Start-Sleep -Seconds 3

Set-Content $file "Modified content - Script Change 2"

Set-Content $file "Original report content"
```

### Important Observation

The displayed script did not contain the complete manually performed sequence.

The lab therefore included both:

- Manual `Set-Content` commands
- Execution of `FileChangeSimulation.ps1`

### Lesson

Always verify a simulation script before executing it:

```powershell
Get-Content "C:\SOC-Lab\FileChangeSimulation.ps1"
```

---

## Issue 5 — Manual File Changes and Scripted Changes Were Both Used

The target file was manually changed using commands such as:

```powershell
Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Modified content - Change 1"
```

and:

```powershell
Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Original report content"
```

The PowerShell script was also executed:

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\SOC-Lab\FileChangeSimulation.ps1"
```

### Lesson

When building the final timeline, manual activity and scripted activity must be treated as separate evidence sources.

---

## Issue 6 — PowerShell Event ID 4104 Was Available but Not Directly Correlated

PowerShell Operational logs contained Event ID 4104 events.

However, the specific 4104 event displayed during the investigation was associated with earlier PowerShell activity.

It did not provide sufficient evidence to directly attribute the target-file modifications to:

`C:\SOC-Lab\FileChangeSimulation.ps1`

### Lesson

The presence of Event ID 4104 in the environment does not automatically mean that every PowerShell action has been recovered.

The analyst must correlate:

- Timestamp
- User
- Process
- Script block
- File path
- Process ID
- Surrounding events

---

## Issue 7 — Avoiding Unsupported Conclusions

The investigation could confirm:

```text
PowerShell executed.
```

It could also confirm:

```text
The target file was intentionally modified during the lab.
```

However, it could not confirm through the recovered Event ID 11 data that every target-file modification was captured.

Therefore, avoid writing:

```text
Sysmon captured all modifications to Important_Report.txt.
```

Instead write:

```text
Sysmon was operational and generated Event ID 11 telemetry, but direct Event ID 11 evidence for every modification of Important_Report.txt was not recovered.
```

---

## Troubleshooting Checklist

- [x] Opened PowerShell with administrative privileges
- [x] Verified Sysmon configuration access
- [x] Verified Sysmon Event ID 1
- [x] Verified Sysmon Event ID 11 was present
- [x] Created the SOC lab directory
- [x] Created the target file
- [x] Modified the target file
- [x] Restored the target file
- [x] Repeated the modification/restoration sequence
- [x] Created the PowerShell simulation script
- [x] Verified the script contents
- [x] Executed the simulation script
- [x] Queried Event ID 23
- [x] Checked PowerShell Event ID 4104 availability
- [ ] Recovered direct Event ID 11 evidence for every target-file modification
- [ ] Recovered direct Event ID 4104 evidence for the exact simulation script
