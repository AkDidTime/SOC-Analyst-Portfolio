# Linux Authentication Log Investigation

## Overview

This investigation focused on reviewing Linux authentication logs and system activity to identify potential signs of suspicious login behaviour. Using Linux Mint 22.3 XFCE, I examined authentication records, login history, system journal entries and SSH configuration to understand how Linux records user activity and how these logs can support a security investigation.

The investigation was conducted in a Linux Mint Live USB environment, allowing me to explore Linux security logs and authentication mechanisms without installing Linux onto the laptop's internal drive.

## Investigation Objectives

- Review Linux authentication logs and user session activity.
- Check for failed login attempts and authentication errors.
- Examine login history and available system logs.
- Check SSH service status and look for evidence of SSH authentication activity.
- Review listening network ports and running processes.
- Assess the findings and identify any areas requiring further investigation.

## Investigation Environment

| Component | Details |
|---|---|
| Operating System | Linux Mint 22.3 (Zena) XFCE |
| Base Distribution | Ubuntu 24.04 (Noble) |
| Kernel | Linux 6.14.0-37-generic |
| Environment | Live USB |
| Tools | Terminal, grep, faillock, last, journalctl, systemctl, ss, ps |

## Investigation Process

### 1. System Environment Review

I began by identifying the Linux distribution, version and kernel information using system commands. This established the environment in which the investigation was conducted.

The system was running Linux Mint 22.3 (Zena) XFCE on an Ubuntu-based distribution.

**Evidence:**
- `Evidence/01-System-Environment-Details.png`
- `Evidence/08-System-Uptime-Details.png`

### 2. Authentication Log Review

I reviewed `/var/log/auth.log` using `grep` to identify authentication-related events, including session activity, administrative commands and potential login failures.

The logs showed a graphical login for the `mint` user, systemd session activity, administrative actions through `pkexec` and scheduled CRON activity.

No clear evidence of failed authentication or suspicious login activity was identified in the entries reviewed.

**Evidence:**
- `Evidence/02-Authentication-Log-Analysis.png`

### 3. Failed Login Checks

I searched the authentication logs for failed passwords, invalid users and authentication failures. I also used `faillock` to check whether any failed-login records were available for the `mint` user.

The command returned no recorded entries. However, this result alone does not confirm that no failed login attempts had occurred, as the available records were limited to the Live USB session.

**Evidence:**
- `Evidence/03-Failed-Login-Checks.png`

### 4. Login History Review

I used `last` to examine the available login and boot records. The output showed one graphical session for the `mint` user and a record of the current boot.

The session was marked `gone - no logout`, indicating that a logout record was not available. This was not treated as suspicious in isolation.

The login history also began at the current boot, limiting the amount of historical activity available for review.

**Evidence:**
- `Evidence/04-Login-History-Analysis.png`
- `Evidence/07-Boot-Record-Limitations.png`

### 5. System Journal Review

I used `journalctl` to review recent system events, boot information and warnings.

Several system and hardware-related warnings were present, including an Intel Management Engine public-key retrieval error. Other warnings related to system components and hardware.

No evidence linked these warnings to malicious activity. They were documented as system issues rather than indicators of compromise.

### 6. SSH Service and Network Port Checks

I checked the status of the SSH service and whether the OpenSSH server package was installed.

The SSH service was inactive, and the OpenSSH server package was not installed. Searches of the available authentication logs also returned no matching SSH authentication events.

I also used `ss -lntup` to review listening network ports and identify services accepting local connections. The results showed several listening services, including local DNS-related and display-related ports.

There was no evidence of SSH login activity in the records reviewed. However, these checks do not rule out other forms of remote access or activity outside the available logs.

**Evidence:**
- `Evidence/05-SSH-Service-Status.png`
- `Evidence/06-Listening-Ports-Check.png`

## Findings

| Area | Findings |
|---|---|
| System environment | Linux Mint 22.3 XFCE running in a Live USB environment |
| Authentication logs | Normal local session and administrative activity observed |
| Failed logins | No clear evidence found in the records reviewed |
| Login history | One graphical session and the current boot recorded |
| System journal | System and hardware-related warnings identified |
| SSH activity | SSH server not installed; service inactive; no matching authentication events found |
| Network ports | Listening local services identified and reviewed |
| Indicators of compromise | No confirmed indicators identified in the available evidence |

## Conclusion

The investigation found no confirmed evidence of suspicious authentication activity in the available logs. The events reviewed were consistent with normal user-session activity, administrative actions and system services.

Although several system warnings were present, the evidence did not establish a connection between these warnings and a security incident. The SSH checks also found no evidence of SSH authentication activity in the records examined.

The main limitation was the use of a Linux Mint Live USB environment. The available logs covered a limited period and did not provide a complete history of previous authentication activity.

The findings therefore apply only to the records reviewed and do not prove that no failed logins or security incidents had occurred.

## Recommended Actions

- Review authentication logs in a persistent Linux environment to build a longer-term record of activity.
- Investigate controlled failed-login attempts to understand how suspicious authentication patterns appear in Linux logs.
- Monitor repeated authentication failures and attempts involving invalid usernames.
- Review listening network services and investigate unexpected services or exposed ports.
- Explore forwarding Linux logs to a SIEM platform for centralised monitoring and correlation.

## Evidence Collected

The investigation evidence is stored in the `Evidence` directory and includes screenshots of system information, authentication log analysis, failed-login checks, login history, SSH service status, listening ports and boot records.

## Skills Demonstrated

- Linux authentication log analysis
- Reviewing login history and user sessions
- Identifying and assessing authentication-related events
- Investigating system journal warnings
- Checking SSH service configuration and status
- Reviewing listening network ports
- Documenting findings and investigation limitations

---

[Return to Investigations](../README.md)
