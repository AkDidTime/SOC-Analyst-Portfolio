# 🔎 Sysmon Process Investigation

## Overview

This investigation examines Windows process creation activity using **Sysmon Event ID 1** to determine whether observed process execution was legitimate or potentially suspicious.

The investigation focuses on process relationships, execution context, command-line activity, and endpoint telemetry to assess the behaviour observed on the system.

## Investigation Objective

The objective of this investigation was to:

- Analyse Sysmon process creation events
- Identify parent-child process relationships
- Examine PowerShell execution
- Review command-line and process context
- Determine whether the observed activity was suspicious
- Identify any potential indicators of compromise (IOCs)
- Document the investigation and final assessment

## Environment

| Component | Details |
|---|---|
| Operating System | Windows 10 Home |
| Monitoring Tool | Sysmon v15.21 |
| Primary Event | Sysmon Event ID 1 — Process Create |
| Log Source | Microsoft-Windows-Sysmon/Operational |
| Analysis Tools | Windows Event Viewer, PowerShell |

## Investigation Workflow

1. Identify relevant Sysmon process creation events
2. Examine the process and execution context
3. Analyse parent-child process relationships
4. Review command-line information
5. Check for suspicious behaviour or indicators of compromise
6. Determine whether the activity is legitimate or suspicious
7. Document the findings and conclusion

---

# 1. Initial Observation

A Sysmon Event ID 1 process creation event was identified for investigation.

The event provided endpoint telemetry including:

- Process image
- Process ID
- Process GUID
- Parent process information
- Command line
- User context
- Integrity level
- Timestamp

The investigation focused on understanding how the observed process was launched and whether its execution context was consistent with normal activity.

---

# 2. Process Relationship Analysis

The investigated process chain was:

```text
explorer.exe
    ↓
powershell.exe
    ↓
conhost.exe
```
This represents a parent-child process relationship.

explorer.exe launched powershell.exe, which subsequently spawned conhost.exe.

Understanding these relationships is useful during endpoint investigations because unusual process chains can indicate suspicious execution.

For example, a document application unexpectedly spawning PowerShell may warrant additional investigation.

In this case, the observed process relationship was consistent with normal interactive activity.

---

# 3. PowerShell Analysis

The PowerShell process was examined using the available Sysmon telemetry.

The investigation considered:

Parent process
Process image
Command line
User context
Integrity level
Execution location
Process relationships

No suspicious command-line arguments or unusual execution context were identified.

The PowerShell activity appeared to originate from normal interactive use through explorer.exe.

---

# 4. Indicator of Compromise Analysis

The available telemetry was reviewed for potential indicators of compromise.

The investigation did not identify:

Suspicious command-line activity
An unusual parent process
Unexpected process relationships
Other obvious indicators of compromise

The observed processes were consistent with legitimate Windows activity.

---

# 5. Findings

The investigation established the following process relationship:
```
explorer.exe
    ↓
powershell.exe
    ↓
conhost.exe
```
The process hierarchy and available execution context did not indicate malicious activity.

The PowerShell execution was assessed as consistent with legitimate interactive activity.

---

# 7. Skills Demonstrated
Sysmon
Windows Event Logs
PowerShell
Process creation analysis
Parent-child process relationships
Command-line analysis
Endpoint telemetry
IOC analysis
Security investigation
Evidence-based assessment
Technical documentation

---

# 6. Conclusion

The observed PowerShell execution was consistent with legitimate interactive activity.

The process tree showed explorer.exe launching PowerShell, which in turn spawned conhost.exe. No suspicious command line, unusual parent process, or other indicators of compromise were identified during the investigation.

The investigation demonstrated how Sysmon process creation telemetry can be used to analyse process relationships and provide endpoint context when assessing potentially suspicious activity.

> [Return To Investigations](../README.md)
