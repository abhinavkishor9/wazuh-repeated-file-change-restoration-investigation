# Lab 58 — Investigation Timeline

## Timeline Overview

This timeline documents the repeated modification and restoration investigation performed against:

`C:\SOC-Lab\Important_Report.txt`

The timeline separates:

- Manual lab activity
- Script creation
- Script execution
- Sysmon telemetry
- PowerShell telemetry
- Missing telemetry

---

## Timeline

| Time | Source | Event | Evidence |
|---|---|---|---|
| 22-08-2026 07:29 | PowerShell | `C:\SOC-Lab` directory created | `New-Item` |
| 22-08-2026 07:29+ | PowerShell | Lab directory checked | `Get-ChildItem` |
| 22-08-2026 07:xx | PowerShell | Target file created | `Important_Report.txt` |
| 22-08-2026 07:xx | PowerShell | First file modification | `Modified content - Change 1` |
| 22-08-2026 07:xx | PowerShell | First restoration | `Original report content` |
| 22-08-2026 07:xx | PowerShell | Second file modification | `Modified content - Change 2` |
| 22-08-2026 07:xx | PowerShell | Second restoration | `Original report content` |
| 22-08-2026 07:xx | PowerShell | Third file modification | `Modified content - Change 3` |
| 22-08-2026 07:xx | PowerShell | Simulation script created | `FileChangeSimulation.ps1` |
| 22-08-2026 07:39:17 local | Sysmon Event ID 1 | PowerShell process created | `powershell.exe`, PID 14700 |
| 22-08-2026 07:39:17 local | Sysmon Event ID 11 | PowerShell temporary file created | `__PSScriptPolicyTest_v42nuax3.b5n.ps1` |
| 22-08-2026 07:xx | PowerShell | Simulation script executed | `FileChangeSimulation.ps1` |
| 22-08-2026 07:xx | Sysmon Event ID 23 | Target-file deletion searched | No matching event |
| 22-08-2026 07:xx | PowerShell | Final file state available | Target file remained part of lab investigation |

---

## Confirmed Sysmon Events

### Event ID 1 — Process Creation

UTC timestamp:

`2026-08-22 02:09:17.338 UTC`

Relevant process:

`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

Process ID:

`14700`

Process GUID:

`{6a3a75f0-04cd-6a89-3504-00000000a101}`

Computer:

`DESKTOP-9MMM37V`

### Interpretation

PowerShell execution was successfully recorded by Sysmon.

---

## Event ID 11 — File Creation

UTC timestamp:

`2026-08-22 02:09:17.810 UTC`

Process:

`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

Process ID:

`14700`

Target file:

`C:\Users\Dell\AppData\Local\Temp\__PSScriptPolicyTest_v42nuax3.b5n.ps1`

### Interpretation

Sysmon Event ID 11 was active and recorded a file creation event associated with PowerShell.

However, the target was a PowerShell temporary file.

It was not:

`C:\SOC-Lab\Important_Report.txt`

---

## Manual File Activity Timeline

The controlled file sequence was:

```text
Original report content
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

These state changes were generated intentionally using `Set-Content`.

---

## Script Execution Timeline

The simulation script was executed using:

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\SOC-Lab\FileChangeSimulation.ps1"
```

The intended relationship was:

```text
powershell.exe
      |
      v
FileChangeSimulation.ps1
      |
      v
Important_Report.txt
      |
      +--> Modification
      |
      +--> Restoration
      |
      +--> Modification
      |
      +--> Restoration
```

---

## Event Correlation

The strongest confirmed telemetry relationship was:

```text
Sysmon Event ID 1
        |
        v
powershell.exe
        |
        | PID 14700
        v
PowerShell activity
        |
        v
Sysmon Event ID 11
        |
        v
PowerShell temporary file
```

The target file activity was separately confirmed through the controlled lab commands.

The recovered logs did not establish a complete Event ID 11 sequence for:

`C:\SOC-Lab\Important_Report.txt`

---

## Event ID 23 Investigation

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

### Timeline Interpretation

No target-file deletion event was recovered.

The lab did not intentionally delete the target file.

The main file operation was:

```text
Set-Content
```

which changes the file contents.

---

## Evidence Gaps

The following evidence was not directly recovered:

```text
Event ID 11 for every modification of Important_Report.txt
```

and:

```text
Event ID 4104 directly showing the FileChangeSimulation.ps1 execution
```

and:

```text
Event ID 23 for Important_Report.txt
```

These gaps should be explicitly documented rather than filled with assumptions.

---

## Final Timeline Assessment

### Confirmed Activity

```text
C:\SOC-Lab created
        |
        v
Important_Report.txt created
        |
        v
File modified
        |
        v
File restored
        |
        v
File modified again
        |
        v
File restored again
        |
        v
File modified again
```

### Confirmed Telemetry

```text
Sysmon Event ID 1
        |
        v
powershell.exe
PID 14700
```

and:

```text
Sysmon Event ID 11
        |
        v
PowerShell temporary file
```

### Missing Direct Evidence

```text
Target-file Event ID 11
        +
Target-file Event ID 23
        +
Direct Event ID 4104 correlation
```

---

## Final Investigation Conclusion

The timeline demonstrates that repeated file modification and restoration was successfully generated during the controlled lab.

PowerShell process execution was recorded by Sysmon.

Sysmon Event ID 11 was also operational and recorded a PowerShell temporary file creation.

However, the recovered telemetry did not provide a complete event-by-event record for `Important_Report.txt`.

The correct DFIR conclusion is:

```text
The file-change activity occurred as part of a controlled lab simulation.

PowerShell execution was observed.

Sysmon was operational.

Direct telemetry for every modification of Important_Report.txt
was not recovered.

No matching Event ID 23 deletion event was identified.
```

---

## DFIR Lesson

The final timeline must distinguish between:

```text
Known Activity
```

```text
Observed Telemetry
```

and:

```text
Telemetry Gaps
```

This prevents the analyst from confusing an action performed during the lab with an action that was actually captured in the available logs.
