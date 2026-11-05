# Elastic SIEM Investigation

**Status:** Completed  
**Platform:** Elastic Security (Elastic Cloud)  
**Endpoint:** Windows 10  
**Focus:** Windows Event Log Analysis | SIEM | Detection Engineering | Authentication Monitoring

## Overview

This project involved setting up Elastic Security in the cloud and connecting a Windows endpoint using Elastic Agent. The aim was to collect Windows security events, investigate authentication activity and create a detection rule to identify repeated failed logon attempts.

I used Windows Security Event IDs 4624 and 4625 to review successful and failed authentication activity. I then created and tested a custom ES|QL detection rule to identify repeated failed logons involving the same account and endpoint.

## Objectives

- Deploy Elastic Agent and connect a Windows endpoint to Elastic Security.
- Configure Windows and System integrations to collect endpoint telemetry.
- Ingest and review Windows Security events.
- Investigate successful and failed authentication activity.
- Create an ES|QL detection rule for repeated failed logons.
- Generate and review an alert through Elastic Security.

## Lab Environment

| Component | Purpose |
|---|---|
| Elastic Security | SIEM platform for log collection, analysis and detection |
| Elastic Cloud | Hosted Elastic deployment |
| Elastic Agent | Collects and forwards endpoint telemetry |
| Windows 10 | Monitored endpoint |
| Windows Integration | Collects Windows event logs, including PowerShell, Sysmon and Defender events |
| System Integration | Collects Windows system logs and metrics |
| ES\|QL | Used to write the failed logon detection rule |

## 1. Deployment and Configuration

I created an Elastic Cloud deployment and configured an agent policy named `Windows-SOC-Lab`.

The Windows and System integrations were added to the policy to collect endpoint telemetry. I downloaded Elastic Agent for Windows x86_64 and installed it on my Windows machine.

After enrolling the agent into Fleet, I checked its status in the Agents section. It appeared as **Healthy**, confirming that it had successfully connected to Elastic Cloud.

**Evidence:**
- `Elastic-Agent-Deployed.png`
- `Elastic-Integration-Details.png`

## 2. Log Collection and Initial Review

Once the agent was running, I opened Discover and used ES|QL queries to review the ingested data.

The collected datasets included:

- `system.security`
- `system.application`
- `system.system`
- `windows.sysmon_operational`
- `windows.windows_defender`
- `windows.powershell`

The `system.security` dataset contained Windows authentication events, which became the focus of the investigation.

An initial review of the authentication events identified:

| Event ID | Description | Events observed |
|---|---|---:|
| 4624 | Successful logon | 99 |
| 4625 | Failed logon | 2 |

These counts were observed in the selected 24-hour search period.

**Evidence:**
- `Elastic-Authentication-Event-Summary.png`
- `Elastic-4624-Successful-Logons-Overview.png`

## 3. Failed Logon Investigation

I filtered the Windows Security events for Event ID 4625 to examine failed authentication attempts.

Both events involved the local test account `SOC-LabUser` on the monitored Windows endpoint.

The event details showed:

| Field | Value |
|---|---|
| Event ID | 4625 |
| Account | SOC-LabUser |
| Logon Type | 2 – Interactive |
| Status | `0xc000006d` |
| SubStatus | `0xc000006a` |
| Caller Process | `svchost.exe` |
| Source Network Address | `127.0.0.1` |
| Authentication Package | Negotiate |

The status and substatus indicated that the logon failed because of an incorrect username or authentication information, with the substatus identifying an incorrect password.

The source address was the local loopback address (`127.0.0.1`), rather than an external IP address.

These failures were deliberately generated as part of a controlled test using `SOC-LabUser`. They were not evidence of an actual external brute-force attack.

**Evidence:**
- `Elastic-4625-Failed-Logons-Overview.png`
- `Elastic-Failed-Logon-Event-JSON.png`

## 4. Successful Logon Review

I also reviewed Event ID 4624 to understand the successful authentication activity recorded on the endpoint.

The events included service logons using the SYSTEM account, alongside other local Windows authentication activity.

The logon types and account information helped distinguish ordinary system activity from the failed attempts involving the test account.

I did not identify evidence of an external source IP in the reviewed records.

**Evidence:**
- `Elastic-4624-Successful-Logons-Overview.png`

## 5. Detection Rule Creation

After reviewing the authentication events, I created a custom ES|QL detection rule in Elastic Security.

The rule was named:

`Multiple Failed Windows Logons - SOC Lab`

Its purpose was to identify two or more failed Windows authentication attempts against the same account on the same host within a five-minute period.

### Detection query

```sql
FROM logs-*
| WHERE event.dataset == "system.security"
  AND event.code == "4625"
  AND @timestamp >= NOW() - 5 minutes
| STATS failed_logons = COUNT(*) BY host.name, user.name
| WHERE failed_logons >= 2
| KEEP host.name, user.name, failed_logons
```

### Rule configuration

| Setting | Value |
|---|---|
| Rule type | ES\|QL |
| Event source | Windows Security |
| Event ID | 4625 |
| Threshold | 2 or more failed logons |
| Grouping | Host and username |
| Detection window in query | 5 minutes |
| Severity | Medium |
| Risk score | 47 |
| Execution interval | 1 minute |

The rule was enabled in Elastic Security and configured to run automatically.

**Evidence:**
- `Elastic-Detection-Rule-Configuration.png`
- `Elastic-Detection-Rule-Creation.png`

## 6. Alert Generation and Review

After creating the detection rule, Elastic Security generated an alert named `Multiple Failed Windows Logons - SOC Lab`.

The alert contained the following information:

| Field | Value |
|---|---|
| Rule | Multiple Failed Windows Logons - SOC Lab |
| Account | SOC-LabUser |
| Host | desktop-fe01e2h |
| Failed logons | 2 |
| Severity | Medium |
| Risk score | 47 |
| Status | Open |

This confirmed that the detection rule had generated an alert containing the two failed logon attempts.

One point to verify is the timing: the displayed source events are timestamped around 15:35, while the alert was created around 16:15. Although the alert contains a count of two, these timestamps do not independently demonstrate that the events fell within the intended five-minute window. The detection's time-window behaviour would need further validation before relying on it in a production environment.

**Evidence:**
- `Elastic-Multiple-Failed-Logons-Alert-Overview.png`
- `Elastic-Failed-Logon-Alert-Details.png`

## Findings

- Successfully deployed Elastic Agent and connected the Windows endpoint to Elastic Cloud.
- Configured Windows and System integrations to collect endpoint telemetry.
- Successfully ingested Windows Security authentication events.
- Reviewed 99 successful logons and 2 failed logons in the selected 24-hour period.
- Identified two controlled failed authentication attempts involving `SOC-LabUser`.
- Confirmed that the failed logons used Logon Type 2 and originated from the local machine.
- Created a custom ES|QL detection rule and observed an alert generated by Elastic Security.

The reviewed evidence did not establish an external brute-force attack or a confirmed system compromise.

## Recommendations

If this were a production environment, I would recommend:

- Investigating repeated failed logons and checking whether they are followed by successful authentication.
- Reviewing the source IP, account, hostname and logon type to establish the origin of suspicious activity.
- Correlating authentication events with additional endpoint and network telemetry.
- Tuning the detection threshold and time window to reduce false positives.
- Validating the rule's time-window behaviour and alert generation before deploying it in a production environment.

## Conclusion

This project gave me practical experience setting up a cloud-hosted SIEM, collecting Windows endpoint telemetry and investigating authentication events through Elastic Security.

I also gained experience writing an ES|QL detection query and reviewing the resulting alert. The investigation demonstrated how Windows Security logs can be used to identify failed authentication activity and how detection rules can help bring relevant events to an analyst's attention.

All testing was carried out on my own Windows endpoint using a controlled local test account. No real attack or compromise was involved.
