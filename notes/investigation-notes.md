# Lab 58 — Investigation Notes

## 1. Investigation Summary

This investigation examined repeated modification and restoration of:

`C:\SOC-Lab\Important_Report.txt`

The activity was intentionally generated on the Windows endpoint using PowerShell.

The investigation focused on determining:

- What file was affected
- What process was involved
- Whether Sysmon captured the activity
- Whether PowerShell telemetry was available
- Whether file deletion telemetry existed
- What could and could not be proven from the available logs

---

## 2. Host Information

```text
Hostname:
DESKTOP-9MMM37V

Observed User:
Dell

Operating System:
Windows

Primary Monitoring:
Sysmon

PowerShell:
Windows PowerShell
```

---

## 3. Lab Directory

The following command was used:

```powershell
New-Item -Path "C:\SOC-Lab" -ItemType Directory -Force
```

The directory was successfully created:

```text
C:\SOC-Lab
```

The directory was then checked using:

```powershell
Get-ChildItem C:\SOC-Lab
```

The directory was initially empty.

---

## 4. Target File

The primary investigation target was:

`C:\SOC-Lab\Important_Report.txt`

The initial content was:

```text
Original report content
```

---

## 5. Manual File Modification

### Modification 1

```powershell
Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Modified content - Change 1"
```

The resulting content was verified as:

```text
Modified content - Change 1
```

### Restoration 1

```powershell
Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Original report content"
```

The resulting content was verified as:

```text
Original report content
```

### Modification 2

```powershell
Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Modified content - Change 2"
```

### Restoration 2

```powershell
Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Original report content"
```

### Modification 3

```powershell
Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Modified content - Change 3"
```

---

## 6. PowerShell Simulation Script

A script was created:

`C:\SOC-Lab\FileChangeSimulation.ps1`

The script targeted:

`C:\SOC-Lab\Important_Report.txt`

The script contained repeated `Set-Content` operations.

The script was executed using:

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\SOC-Lab\FileChangeSimulation.ps1"
```

---

## 7. Sysmon Event ID 1 Investigation

Sysmon Event ID 1 was available.

The relevant event contained:

```text
Event ID:
1

Event:
Process Create

Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

Process ID:
14700

Process GUID:
{6a3a75f0-04cd-6a89-3504-00000000a101}

UTC Time:
2026-08-22 02:09:17.338 UTC

Computer:
DESKTOP-9MMM37V
```

### Interpretation

This establishes that Windows PowerShell executed on the endpoint.

The process was:

`powershell.exe`

with:

`PID 14700`

This provides process-level evidence for the investigation.

---

## 8. Sysmon Event ID 11 Investigation

Sysmon Event ID 11 was available.

An observed Event ID 11 event contained:

```text
Event ID:
11

Event:
File created

UtcTime:
2026-08-22 02:09:17.810 UTC

Process:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

Process ID:
14700

TargetFilename:
C:\Users\Dell\AppData\Local\Temp\__PSScriptPolicyTest_v42nuax3.b5n.ps1
```

### Interpretation

This confirms that Sysmon was generating Event ID 11 events.

However, the target of this particular event was:

`C:\Users\Dell\AppData\Local\Temp\__PSScriptPolicyTest_v42nuax3.b5n.ps1`

and not:

`C:\SOC-Lab\Important_Report.txt`

Therefore, this event should not be interpreted as direct evidence that Sysmon captured a modification of the target file.

---

## 9. Target File Event ID 11 Search

The following query was used:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName="Microsoft-Windows-Sysmon/Operational"
    Id=11
} -MaxEvents 100 |
Where-Object {$_.Message -match "Important_Report.txt"} |
Select-Object TimeCreated, Id, Message
```

Result:

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

### Assessment

No matching Event ID 11 record for:

`C:\SOC-Lab\Important_Report.txt`

was recovered from the query.

This does not establish that the file was never modified.

The file was intentionally modified during the lab.

It establishes only that the queried Event ID 11 telemetry did not provide a matching target-file event.

---

## 10. Sysmon Event ID 23 Investigation

The following query was executed:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName="Microsoft-Windows-Sysmon/Operational"
    Id=23
} -MaxEvents 100 |
Where-Object {$_.Message -match "Important_Report.txt"} |
Select-Object TimeCreated, Id, Message
```

Result:

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

### Assessment

No Event ID 23 record for the target file was identified.

The simulation used `Set-Content`, which changes the contents of the existing file.

The lab did not intentionally perform a delete operation against the target file.

---

## 11. PowerShell Event ID 4104

PowerShell Operational logging contained Event ID 4104 records.

The environment showed:

```text
Log:
Microsoft-Windows-PowerShell/Operational

Event ID:
4104

Number of events:
444
```

This confirms that PowerShell Script Block Logging was available.

However, the specific 4104 event displayed during the investigation was from an earlier date and contained unrelated PowerShell activity.

Therefore, it was not used as direct evidence for the `FileChangeSimulation.ps1` execution.

---

## 12. Evidence Matrix

| Evidence | Status | Interpretation |
|---|---|---|
| `C:\SOC-Lab` created | Confirmed | Lab preparation |
| `Important_Report.txt` created | Confirmed | Lab preparation |
| File modified | Confirmed | Controlled lab action |
| File restored | Confirmed | Controlled lab action |
| File modified repeatedly | Confirmed | Controlled lab action |
| PowerShell executed | Confirmed | Sysmon Event ID 1 |
| Sysmon Event ID 11 operational | Confirmed | FileCreate telemetry available |
| Event ID 11 for target file | Not recovered | No matching result |
| Event ID 23 for target file | Not recovered | No matching result |
| PowerShell 4104 available | Confirmed | Script Block Logging enabled |
| 4104 for exact lab script | Not established | Retrieved event was unrelated |

---

## 13. Process Correlation

The confirmed process telemetry was:

```text
powershell.exe
    |
    +-- PID: 14700
    |
    +-- Process GUID:
        {6a3a75f0-04cd-6a89-3504-00000000a101}
```

The PowerShell process was involved in the controlled lab actions.

However, because the target-file Event ID 11 record was not recovered, the investigation does not claim that the specific Sysmon Event ID 11 event captured each `Set-Content` operation.

---

## 14. Investigation Conclusion

The investigation established:

1. The target file was intentionally modified and restored.
2. PowerShell was used during the lab.
3. Sysmon Event ID 1 captured PowerShell process creation.
4. Sysmon Event ID 11 was operational.
5. The recovered Event ID 11 event involved a PowerShell temporary file.
6. No matching Event ID 11 event for `Important_Report.txt` was recovered.
7. No matching Event ID 23 event for `Important_Report.txt` was recovered.
8. PowerShell Event ID 4104 logging was available, but the retrieved example did not directly represent this lab activity.

---

## 15. Final Classification

```text
Classification:
Benign / Controlled Lab Simulation
```

The activity itself was intentionally generated.

The same behavior in a production environment would require additional investigation if it involved an unknown process, unauthorized user, multiple files, suspicious command lines, or other indicators of compromise.

---

## 16. SOC Analyst Lesson

The most important lesson from this lab is:

```text
Observed activity
        !=
Complete telemetry
```

An analyst must distinguish between:

```text
Known lab actions
```

```text
Observed log evidence
```

and:

```text
Missing telemetry
```

This prevents unsupported conclusions and improves the quality of SOC investigation reports.
