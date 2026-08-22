# Lab 58 — Repeated File Change and Restoration Investigation

## Overview

This lab investigates repeated modification and restoration of a file on a Windows endpoint.

The activity was intentionally generated using PowerShell against:

`C:\SOC-Lab\Important_Report.txt`

The investigation focuses on correlating file activity with process execution and Windows telemetry to determine:

- Which file was affected
- How the file was modified and restored
- Which process was involved
- Which PowerShell script was executed
- Whether Sysmon captured the expected file activity
- What telemetry was available
- What telemetry was missing
- How to build a defensible investigation timeline

---

## Lab Scenario

A monitored Windows workstation shows repeated changes to the same file over a short period.

The SOC analyst must determine whether the activity represents:

- Legitimate administrative activity
- Expected application behavior
- Automated file manipulation
- Unauthorized file tampering
- Potential ransomware-like behavior

The investigation begins with:

`C:\SOC-Lab\Important_Report.txt`

The file was intentionally modified and restored multiple times to create a controlled investigation scenario.

---

## Objectives

By completing this lab, the analyst should be able to:

1. Identify a repeatedly modified file.
2. Identify the process associated with the activity.
3. Correlate PowerShell execution with file activity.
4. Investigate Sysmon Event ID 1.
5. Investigate Sysmon Event ID 11.
6. Investigate Sysmon Event ID 23.
7. Review PowerShell Event ID 4104 when available.
8. Build a timeline from multiple telemetry sources.
9. Identify telemetry gaps.
10. Distinguish observed evidence from manually generated lab activity.
11. Avoid incorrectly classifying repeated file modification as ransomware without sufficient evidence.

---

## Environment

| Component | Details |
|---|---|
| Operating System | Windows |
| Hostname | DESKTOP-9MMM37V |
| User observed | Dell |
| Monitoring | Sysmon |
| PowerShell | Windows PowerShell |
| Log Sources | Sysmon / PowerShell Operational |
| Lab Directory | `C:\SOC-Lab` |
| Target File | `Important_Report.txt` |
| Simulation Script | `FileChangeSimulation.ps1` |

---

## Relevant Telemetry

### Sysmon

| Event ID | Purpose |
|---|---|
| 1 | Process Creation |
| 11 | File Create |
| 23 | File Delete |
| 26 | File Delete Logged |

### PowerShell

| Event ID | Purpose |
|---|---|
| 4104 | PowerShell Script Block Logging |

---

## Lab Setup

The lab directory was created using:

```powershell
New-Item -Path "C:\SOC-Lab" -ItemType Directory -Force
```

The resulting directory was:

```text
C:\SOC-Lab
```

The target file was:

```text
C:\SOC-Lab\Important_Report.txt
```

---

## Initial File State

The original file content was:

```text
Original report content
```

The file was then intentionally modified and restored multiple times.

---

## Repeated File Modification Sequence

The controlled activity followed this general sequence:

```text
Original content
        |
        v
Modified content - Change 1
        |
        v
Original report content
        |
        v
Modified content - Change 2
        |
        v
Original report content
        |
        v
Modified content - Change 3
```

The commands used included:

```powershell
Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Modified content - Change 1"

Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Original report content"

Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Modified content - Change 2"

Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Original report content"

Set-Content -Path "C:\SOC-Lab\Important_Report.txt" -Value "Modified content - Change 3"
```

---

## PowerShell Simulation Script

A PowerShell simulation script was created at:

`C:\SOC-Lab\FileChangeSimulation.ps1`

The script targeted:

`C:\SOC-Lab\Important_Report.txt`

The script was executed using:

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\SOC-Lab\FileChangeSimulation.ps1"
```

The script used repeated `Set-Content` operations to modify and restore the target file.

---

## Sysmon Process Creation Evidence

Sysmon Event ID 1 was successfully observed.

The relevant process was:

`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

Important values observed:

```text
Event ID:
1

Process ID:
14700

Process GUID:
{6a3a75f0-04cd-6a89-3504-00000000a101}

UTC Time:
2026-08-22 02:09:17.338 UTC

Computer:
DESKTOP-9MMM37V
```

This confirms that PowerShell was executed on the endpoint.

---

## Sysmon Event ID 11 Evidence

Sysmon Event ID 11 was present in the environment.

An observed Event ID 11 event showed:

```text
Event ID:
11

UtcTime:
2026-08-22 02:09:17.810 UTC

Process:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

Process ID:
14700

TargetFilename:
C:\Users\Dell\AppData\Local\Temp\__PSScriptPolicyTest_v42nuax3.b5n.ps1
```

This confirms that Sysmon was generating Event ID 11 telemetry.

However, the recovered Event ID 11 event was for a PowerShell temporary file and not for:

`C:\SOC-Lab\Important_Report.txt`

Therefore, the investigation does not claim that Sysmon Event ID 11 captured every modification of the target file.

---

## Sysmon Event ID 23 Investigation

The following query was performed:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName="Microsoft-Windows-Sysmon/Operational"
    Id=23
} -MaxEvents 100 |
Where-Object {$_.Message -match "Important_Report.txt"} |
Select-Object TimeCreated, Id, Message
```

The result was:

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

No matching Sysmon Event ID 23 event for `Important_Report.txt` was identified.

The absence of Event ID 23 does not prove that the file could never have been deleted. The lab primarily used `Set-Content`, which modifies file contents rather than explicitly deleting the file.

---

## PowerShell Event ID 4104

PowerShell Operational logging was available.

The environment contained Event ID 4104 records.

However, the 4104 event displayed during the investigation was associated with earlier PowerShell activity and did not provide sufficient evidence to directly attribute the specific `Important_Report.txt` changes to the lab script.

Therefore, the 4104 evidence is treated as confirmation that PowerShell Script Block Logging was available, rather than direct proof of every action performed by the simulation script.

---

## Investigation Findings

### Finding 1 — Repeated file modification was successfully generated

The target file was repeatedly modified and restored during the controlled lab.

### Finding 2 — PowerShell execution was observed

Sysmon Event ID 1 recorded `powershell.exe` with Process ID `14700`.

### Finding 3 — Sysmon Event ID 11 was operational

Sysmon recorded a FileCreate event for a PowerShell temporary file.

### Finding 4 — Direct Event ID 11 evidence for the target file was not recovered

No matching Event ID 11 record for `Important_Report.txt` was identified in the queried results.

### Finding 5 — No Event ID 23 evidence for the target file was recovered

The Event ID 23 query returned no matching result.

### Finding 6 — The activity was a controlled lab simulation

The file changes were intentionally generated and are therefore classified as benign lab activity.

---

## Security Assessment

The lab activity itself was:

**Classification: Benign / Controlled Simulation**

However, the same pattern could be suspicious in a production environment.

Repeated modification and restoration should receive additional investigation when:

- An unknown process performs the changes.
- An unauthorized account performs the activity.
- Multiple sensitive files are affected.
- Files are modified rapidly.
- File extensions are changed.
- Backup or shadow-copy mechanisms are manipulated.
- Network activity occurs around the same time.
- The process has suspicious command-line arguments.
- The parent process is unusual.
- The activity is inconsistent with the user's normal behavior.

---

## Investigation Model

A SOC analyst should correlate:

```text
File
  |
  v
Process
  |
  v
User
  |
  v
Parent Process
  |
  v
Command Line
  |
  v
PowerShell Activity
  |
  v
Other Files
  |
  v
Network Activity
  |
  v
Timeline
```

---

## Key SOC Lesson

A file-change investigation should not rely on a single event.

The analyst must distinguish between:

```text
Activity that actually occurred
```

and:

```text
Activity that was captured by available telemetry
```

In this lab, the file modifications were known to have occurred because they were intentionally performed.

However, the recovered Sysmon data did not provide a complete Event ID 11 record for every target-file modification.

This is an important DFIR lesson because an absence of telemetry is not automatically proof that the activity did not occur.

---

## Conclusion

This lab demonstrated how repeated file modification and restoration can be investigated using PowerShell and Windows telemetry.

The investigation successfully established PowerShell process execution and confirmed that Sysmon Event ID 11 telemetry was operational.

At the same time, the investigation demonstrated a telemetry limitation: direct Sysmon evidence for every modification of `Important_Report.txt` was not recovered.

The final assessment therefore distinguishes between:

- Confirmed lab activity
- Observed telemetry
- Missing telemetry
- Investigation conclusions

This approach produces a more accurate and defensible SOC/DFIR investigation.
