# Lab 58 — Investigation Timeline

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

