# wazuh-repeated-file-change-restoration-investigation
## Overview
A monitored Windows workstation shows repeated changes to the same file over a short period of time. The file appears to be modified, restored, and modified again.

The SOC analyst receives an alert indicating unusual file activity and must determine whether the behavior was legitimate or suspicious.


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

A Windows workstation contains a potentially important file:

C:\SOC-Lab\Important_Report.txt

During monitoring, the file is observed being changed multiple times and then restored to its previous content. The repeated activity occurs over a short period and is associated with PowerShell execution.

The SOC analyst is tasked with investigating the activity and determining whether the repeated file changes represent legitimate administrative activity, an automated process, or potentially suspicious file tampering.

The analyst must correlate Windows and Sysmon telemetry to understand what happened on the endpoint.

The analyst must determine:

- Which file was repeatedly modified?
- What were the different file states during the activity?
- Which process performed or initiated the activity?
- Was PowerShell involved?
- What Sysmon events were generated?
- Did Sysmon record the file changes?
- Was any file deletion activity observed?
- Can the activity be correlated with PowerShell Script Block Logging?
- Are there any gaps in the available telemetry?
- Does the evidence indicate malicious activity or controlled/legitimate activity?

---

## Objectives

- Investigate repeated file modifications on a Windows endpoint.
- Identify the affected file and document its changing contents.
- Trace file activity back to the responsible process using Sysmon telemetry.
- Analyze Sysmon Event ID 1 to identify PowerShell process creation.
- Analyze Sysmon Event ID 11 for file creation activity associated with the process.
- Check Sysmon Event ID 23 for possible file deletion activity.
- Review PowerShell Event ID 4104 to determine whether script-level evidence is available.
- Correlate timestamps, process IDs, file paths, and process information across multiple log sources.
- Identify telemetry gaps when expected file activity is not present in the collected logs.
- Distinguish between confirmed evidence and assumptions when building the investigation timeline.
- Determine the security significance of repeated file modification and restoration.
- Produce a defensible SOC investigation conclusion based only on the available evidence.

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

