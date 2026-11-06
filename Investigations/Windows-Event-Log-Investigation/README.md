# Windows Event Log Investigation

## Overview

This investigation focused on reviewing Windows Security Event Logs to understand authentication activity on a Windows 10 endpoint. The aim was to identify successful and failed logon events, examine different logon types, and determine whether any activity appeared unusual.

The investigation used **Windows Event Viewer** and **PowerShell** to examine Security Event IDs `4624` and `4625`, with particular attention to the account, logon type, process information and source network address recorded in each event.

## Objectives

- Review successful logon events (Event ID `4624`).
- Identify and understand different Windows logon types.
- Review failed logon events (Event ID `4625`).
- Examine the processes associated with selected authentication events.
- Identify any unusual authentication activity within the available logs.
- Document findings and limitations of the investigation.

## Investigation Environment

| Component | Details |
|---|---|
| Operating System | Windows 10 Home |
| Log Source | Windows Security Event Log |
| Tools | Windows Event Viewer, PowerShell |
| Event IDs | 4624, 4625 |
| Investigation Period | 8 September 2026 – 8 October 2026 |

## Methodology

### 1. Reviewing Successful Logons – Event ID 4624

Event ID `4624` records a successful logon session. I reviewed these events to understand which accounts were logging on, how the sessions were created, and which processes were involved.

PowerShell was used to retrieve and examine the most recent successful logon events:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} -MaxEvents 200
```

I also examined the event's XML data to review individual fields, including the target account, logon type, process name and source network address.

```powershell
$Event = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} -MaxEvents 1

[xml]$Xml = $Event.ToXml()

$Xml.Event.EventData.Data |
    Select-Object Name, '#text'
```

### 2. Analysing Logon Types

Logon types provide context about how a successful session was created. I reviewed the logon types recorded in the available events and focused on the following:

| Logon Type | Description |
|---|---|
| 5 | Service – a service was started by the Service Control Manager. |
| 7 | Unlock – an existing workstation session was unlocked. |
| 11 | Cached Interactive – a user logged on using credentials stored locally on the computer. |

The review also considered whether network logons (Type 3) or remote interactive logons (Type 10) appeared in the 200 successful logon events examined.

### 3. Reviewing Failed Logons – Event ID 4625

Event ID `4625` records a failed logon attempt. These events can help identify repeated authentication failures, incorrect credentials or potentially suspicious access attempts.

I used PowerShell to check the Security log for failed logon events:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
}
```

The query returned no Event ID `4625` records in the available Security log at the time of the investigation.

## Findings

### 1. Service Logon – Logon Type 5

![Service Logon Evidence](Evidence/01-4624-service-logon.png)

A successful logon event was identified with Logon Type `5`, associated with the `SYSTEM` account and the `services.exe` process.

This is consistent with normal Windows service activity. The Service Control Manager starts services under their configured accounts, and these activities can generate successful service logon events.

No suspicious behaviour was identified from the event reviewed.

### 2. Logon Type Analysis

![Logon Type Analysis Evidence](Evidence/02-4624-logon-type-analysis.png)

The review identified several successful logon types:

- **Type 5 – Service:** Associated with Windows service activity.
- **Type 7 – Unlock:** Associated with unlocking an existing workstation session.
- **Type 11 – Cached Interactive:** Associated with a logon using locally cached credentials.

Across the 200 successful logon events examined, no Logon Type `3` (Network) or Logon Type `10` (Remote Interactive) events were identified.

This means no network or Remote Desktop logons of those types were present in the sample reviewed. It does not establish that such activity never occurred outside the available records or sample.

### 3. Failed Logon Review – Event ID 4625

![Failed Logon Query Evidence](Evidence/03-4625-no-failed-logons.png)

The Security log query returned no Event ID `4625` records during the investigation.

This indicates that no failed logon events were available in the retained log data examined. It does not prove that failed authentication attempts never occurred, as event availability depends on audit configuration, log retention and the period covered by the records.

### 4. Cached Interactive Logon – Logon Type 11

![Cached Interactive Logon Evidence](Evidence/04-4624-cached-interactive.png)

A successful logon event was identified with Logon Type `11`, associated with the `svchost.exe` process and the local loopback address `127.0.0.1`.

Logon Type `11` represents a Cached Interactive logon. Windows uses locally stored credentials for this type of logon rather than contacting a domain controller to verify them.

The recorded address `127.0.0.1` is the loopback address, which refers to the local computer itself. It is not a public IP address or an address identifying an external device on the local network.

The event was consistent with local Windows authentication activity, and no suspicious indicators were identified from the fields reviewed.

## Conclusion

The investigation reviewed Windows Security Event IDs `4624` and `4625` to assess authentication activity on the endpoint.

The successful logon events examined included service logons, workstation unlocks and cached interactive logons. The reviewed service and cached interactive events were consistent with normal Windows activity.

No Event ID `4625` records were returned by the failed logon query, and no Logon Type `3` or `10` events were identified within the 200 successful logon events examined.

Based on the available evidence, **no clear indicators of suspicious authentication activity were identified during this investigation**.

However, this conclusion is limited to the Security log records available and the events examined. The absence of failed logon events or network logons in the reviewed data does not prove that no such activity occurred.

## Recommendations

- Ensure successful and failed logon auditing is appropriately configured.
- Review Event IDs `4624` and `4625` regularly to monitor authentication activity.
- Investigate repeated failed logons, unexpected account activity and unusual logon types.
- Pay particular attention to unexpected network logons (Type 3) and Remote Interactive logons (Type 10).
- Maintain appropriate Security log retention so that historical authentication activity remains available for investigation.

## References

- [Microsoft – Audit Logon Events](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/basic-audit-logon-events)
- [Microsoft – Event ID 4624](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624)

> [Return To Investigations](../README.md)
