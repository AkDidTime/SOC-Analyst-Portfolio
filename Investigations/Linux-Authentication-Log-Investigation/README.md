# Linux Authentication Log Investigation

## Overview

This investigation focused on reviewing Linux authentication logs and system activity to identify any signs of suspicious login behaviour. Using Linux Mint 22.3 XFCE, I examined authentication records, login history, system journal entries and SSH status to understand how Linux records user activity and how these logs can support a security investigation.

## Investigation Objectives

- Review Linux authentication logs and user session activity.
- Check for failed login attempts and authentication errors.
- Examine login history and available system logs.
- Check SSH service status and look for evidence of SSH authentication activity.
- Assess the findings and identify any areas requiring further investigation.

## Investigation Process

### 1. Authentication Log Review

I reviewed `/var/log/auth.log` using `grep` to identify authentication-related events, including session activity, administrative commands and potential login failures.

The logs showed a graphical login for the `mint` user, systemd session activity, administrative actions through `pkexec` and a scheduled CRON task. No clear evidence of failed authentication or suspicious login activity was identified in the entries reviewed.

### 2. Failed Login Checks

I searched the authentication logs for failed passwords, invalid users and authentication failures. I also used `faillock` to check whether any failed-login records were available for the `mint` user.

The command returned no recorded entries. However, this result alone does not confirm that no failed login attempts had occurred, as the available records were limited to the Live USB session.

### 3. Login History Review

I used `last` to examine the available login and boot records. The output showed one graphical session for the `mint` user and a record of the current boot.

The session was marked `gone - no logout`, indicating that a logout record was not available. This was not treated as suspicious in isolation. The login history also began at the current boot, limiting the amount of activity that could be reviewed.

### 4. System Journal Review

I used `journalctl` to review recent system events, boot information and warnings. Several system and hardware-related warnings were present, including an Intel Management Engine public-key retrieval error.

No evidence linked these warnings to malicious activity. They were documented as system issues rather than indicators of compromise.

### 5. SSH Service Check

I checked the status of the SSH service and whether the OpenSSH server package was installed. The service was inactive, and the server package was not installed. Searches of the available authentication logs also returned no matching SSH authentication events.

This meant there was no evidence of SSH login activity in the records reviewed. However, the result does not rule out other forms of remote access or activity outside the available logs.

## Findings

| Area | Findings |
|---|---|
| Authentication logs | Normal local session and administrative activity observed |
| Failed logins | No clear evidence found in the records reviewed |
| Login history | One graphical session and the current boot recorded |
| System journal | System and hardware-related warnings identified |
| SSH activity | SSH server not installed; no matching authentication events found |
| Indicators of compromise | No confirmed indicators identified in the available evidence |

## Conclusion

The investigation found no confirmed evidence of suspicious authentication activity in the available logs. The events reviewed were consistent with normal user-session activity, administrative actions and system services.

Although several system warnings were present, the evidence did not establish a connection between these warnings and a security incident.

The main limitation was the use of a Linux Mint Live USB environment. The available logs covered a limited period and did not provide a complete history of previous authentication activity. The findings therefore apply only to the records reviewed and do not prove that no failed logins or security incidents had occurred.

## Recommended Actions

- Review authentication logs in a persistent Linux environment to build a longer-term record of activity.
- Investigate controlled failed-login attempts to understand how suspicious authentication patterns appear in Linux logs.
- Monitor repeated authentication failures and attempts involving invalid usernames.
- Explore forwarding Linux logs to a SIEM platform for centralised monitoring and correlation.

## Evidence Collected

Screenshots of the terminal commands and relevant outputs are stored in the `evidence` folder.

## Skills Demonstrated

- Linux authentication log analysis
- Reviewing login history and user sessions
- Identifying and assessing authentication-related events
- Investigating system journal warnings
- Checking SSH service configuration and status
- Documenting findings and investigation limitations
