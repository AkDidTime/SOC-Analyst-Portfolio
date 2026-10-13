# Brute Force Investigation

## Overview

This investigation explores how failed Windows logon attempts can be identified and analysed using Windows Security logs. I used a local test account to generate controlled authentication failures and examined the resulting events in Event Viewer and PowerShell.

The aim was to understand how failed logons are recorded, identify useful fields in the logs, and look at the timing of events to see how this information could help investigate a suspected brute-force attack.

## Objectives

- Identify failed logon attempts using Windows Security Event ID `4625`.
- Examine account details, logon type, failure status codes and process information.
- Use PowerShell to filter events and build a timeline of failed logons.
- Calculate the time between consecutive event records.
- Understand how repeated authentication failures could be investigated in a SOC environment.

## Tools Used

- **Windows Event Viewer** — to review Security logs and inspect failed logon events.
- **Windows PowerShell** — to filter events, extract relevant information and calculate event intervals.
- **Windows local user account** — to perform controlled authentication tests.

## Investigation Process

### 1. Preparing the test

I created a local account named `SOC-LabUser` for testing authentication activity. I confirmed that the account could sign in successfully before carrying out the failed logon test.

### 2. Reviewing failed logons

I examined Windows Security Event ID `4625`, which is generated when a logon attempt fails. I filtered the events for the test account and reviewed the details recorded by Windows.

The events included the target account, logon type, failure status codes, caller process and source network address.

### 3. Analysing the event details

The failed logon records contained the following values:

| Field | Observed value |
|---|---|
| Event ID | `4625` |
| Target account | `SOC-LabUser` |
| Logon Type | `2` — Interactive |
| Status | `0xC000006D` |
| Substatus | `0xC000006A` |
| Caller process | `C:\Windows\System32\svchost.exe` |
| Source address | `127.0.0.1` |

The status codes were consistent with an incorrect-password failure. The loopback address and other event details were consistent with the local authentication test.

### 4. Building a timeline with PowerShell

I used PowerShell to filter failed logon events for `SOC-LabUser`, sort them by timestamp and calculate the intervals between consecutive event records.

Three records appeared at `11:08:20`, `11:08:21` and `11:08:22` on 13 October 2026. The intervals were approximately 0.79 and 0.80 seconds.

These records followed a single deliberate incorrect-password test. I treated them as separate log entries rather than assuming they represented three independent password attempts.

## Findings

- Windows recorded the controlled failed logons as Event ID `4625`.
- The event details provided useful information for investigating the authentication failure.
- PowerShell made it possible to filter the relevant events and examine their timing.
- Multiple records appeared within a short period, demonstrating why event correlation and context are important when reviewing authentication activity.
- The test account had also generated an earlier failed logon after I accidentally mistyped its password.

## Conclusion

This investigation gave me practical experience using Windows Security logs and PowerShell to examine failed authentication activity. I was able to identify relevant event fields, interpret the failure status codes and build a timeline of the recorded events.

Although the exercise demonstrated techniques that can support a brute-force investigation, the results came from controlled local testing and do not confirm that a real brute-force attack took place. In a real SOC investigation, I would look for repeated independent failures, affected accounts, source systems and any subsequent successful logons before deciding whether the activity was suspicious.

## Recommended Further Investigation

- Check for repeated failed logons across multiple accounts.
- Identify whether failures originate from a common source.
- Correlate failed logons with successful authentication events.
- Establish whether the activity matches normal user behaviour or a potential attack pattern.
- Explore how SIEM tools can detect and alert on suspicious authentication patterns.

## Evidence

Screenshots and supporting evidence will be added to the `evidence/` directory.

- Failed logon events in Event Viewer
- Detailed Event ID `4625` record
- PowerShell event timeline and interval analysis
