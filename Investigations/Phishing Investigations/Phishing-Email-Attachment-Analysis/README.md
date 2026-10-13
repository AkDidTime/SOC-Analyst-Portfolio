# Phishing Investigation — Coinbase-Themed Email

## Overview

This investigation looks at a suspicious email containing a Word document named `Coinbase -15392.docx`. I used PhishTool to review the email and attachment, then used VirusTotal to check the file's reputation and investigate its network activity.

The main aim was to understand why the email was suspicious, identify any relevant indicators of compromise (IOCs), and see what could be learned from the sandbox analysis.

**Verdict:** Malicious or highly suspicious, with multiple phishing-related detections.

## Tools Used

* **PhishTool** — Email and attachment analysis
* **VirusTotal** — File reputation, URL analysis and sandbox behaviour

**Sample source:** [Phishing Investigation Lab — sample-1001.eml](https://github.com/PattonJL/phishing-investigation-lab/blob/main/samples/sample-1001.eml)

---

## 1. Email Analysis

I started by looking at the sender details and other email headers in PhishTool.

| Field           | Value                                                  |
| --------------- | ------------------------------------------------------ |
| Display name    | `support@mail.coinbase.com`                            |
| Sender address  | `mssggeauthencti-cbspprt-325937197367@medisept.com.au` |
| Recipient       | `noreply-KHWG3@mail.assistancehelp.com`                |
| Email timestamp | 26 July 2023, 01:28:42 UTC                             |
| Reply-To        | None recorded                                          |
| SPF             | No record shown                                        |
| DKIM            | No signatures found                                    |
| DMARC           | No record shown                                        |

### What I found

The display name suggests the email came from Coinbase support, but the actual sender address uses the `medisept.com.au` domain. This mismatch raised my suspicion because the sender's identity did not match the organisation they appeared to represent.

The email also had no DKIM signature, and PhishTool showed no SPF or DMARC records in the available analysis. These findings do not prove phishing on their own, but they provided further reasons to investigate the attachment.

The email body appeared blank or contained very little text in PhishTool. I could not determine whether this was intentional or caused by how the message was displayed.

## 2. Attachment Analysis

The email contained a Word document named `Coinbase -15392.docx`. I checked its file details and used its SHA-256 hash to look up the same file in VirusTotal.

| Property  | Value                                                              |
| --------- | ------------------------------------------------------------------ |
| Filename  | `Coinbase -15392.docx`                                             |
| File type | Microsoft Word DOCX                                                |
| File size | 17.21 KB                                                           |
| MD5       | `468818a4c2ee48f8990988d0b8acdbcc`                                 |
| SHA-1     | `345a8699708f8c25bfa68358d440a064d3557257`                         |
| SHA-256   | `a1790a4e09be26f1824f27118b12547f5ea097883f15029532bfc2d1d6610fc4` |
| Macros    | None detected by PhishTool                                         |

### VirusTotal results

At the time I checked, VirusTotal showed **22 detections out of 65 security engines**.

Some vendors, including Microsoft, ESET, Alibaba, Ikarus and Bitdefender, classified the file as phishing-related or Trojan-like. Other engines did not detect anything.

The different results show why it is useful to compare multiple security engines rather than rely on a single verdict. The number of detections and the types of labels strongly suggested that the document was malicious or highly suspicious.

Although PhishTool did not identify any macros, that did not mean the document was safe.

## 3. URL and Network Analysis

I then checked the file's VirusTotal relations and sandbox network activity for any URLs or domains worth investigating.

The analysis identified this shortened URL:

`http://lnkd.in/gun8iQd6`

VirusTotal's Behaviour section recorded a GET request to this URL that returned an HTTP `301` response. This means the server returned a redirect response, although the available report did not show the complete redirect sequence.

Another hostname appeared in the DNS results:

`antibotpw.162-240-106-9.cprapid.com`

Multiple security vendors had flagged this hostname as phishing-related or malicious in the earlier analysis. The sandbox also recorded TCP traffic to `162.240.106.9` over port 443.

Several Google-related domains also appeared in the DNS results. I did not treat these as malicious simply because they were present, as they could have been related to browser activity.

### What this tells me

The shortened URL and the flagged hostname were worth investigating because they appeared alongside a document that already had multiple security-vendor detections.

However, the available report contained only one HTTP request. I could not confirm the full redirect chain or establish whether the suspicious server returned a malicious payload.

## 4. Sandbox Behaviour

I reviewed the Behaviour section in VirusTotal to see what happened during the sandbox run.

### Process activity

The report included processes such as:

* `WINWORD.EXE`
* `explorer.exe`
* `services.exe`
* `svchost.exe`
* `chrome.exe`
* `wmiadap.exe`

A Word process was shown opening a file named `design.docx`, and a Chrome command referenced a URL containing the shortened link.

These entries showed Word and browser activity during the analysis, but the process information available to me did not establish every parent-child relationship. I therefore could not confirm that Word directly launched Chrome or that the document successfully executed a malicious payload.

### Sigma rule matches

VirusTotal also reported matches for rules relating to:

* Changing PowerShell execution policies
* Suspicious creation of an Office COM object
* One executable creating another executable

I treated these as leads rather than proof of malicious activity. The available results did not provide enough context to confirm the exact behaviour behind each match.

### Dropped files

The sandbox listed several files and artefacts, including:

* `AutoRecovery save of design.asd`
* `design.docx.LNK`
* `design.docx.lnk`
* `Coinbase -BQN63.LNK`
* `Normal.dotm (copy)`
* `~WRD0000.tmp`
* Multiple Chrome cache entries

Some of these appear consistent with temporary files or artefacts generated by Word and Chrome. The shortcut files were worth noting because shortcuts can point to files or commands, but I did not have enough evidence to classify each one as malicious.

The sandbox reported hundreds of additional artefacts, so I focused on the entries most relevant to the investigation rather than assuming every dropped file was suspicious.

## 5. Indicators of Compromise (IOCs)

These are the main indicators I identified during the investigation.

| Type                | Indicator                                                          |
| ------------------- | ------------------------------------------------------------------ |
| SHA-256             | `a1790a4e09be26f1824f27118b12547f5ea097883f15029532bfc2d1d6610fc4` |
| MD5                 | `468818a4c2ee48f8990988d0b8acdbcc`                                 |
| Attachment          | `Coinbase -15392.docx`                                             |
| Sender address      | `mssggeauthencti-cbspprt-325937197367@medisept.com.au`             |
| Shortened URL       | `http://lnkd.in/gun8iQd6`                                          |
| Suspicious hostname | `antibotpw.162-240-106-9.cprapid.com`                              |
| Associated IP       | `162.240.106.9`                                                    |

These indicators should be used in context. For example, `lnkd.in` is a shared URL-shortening domain, so the evidence supports investigating the specific URL rather than assuming the entire domain is malicious.

## 6. MITRE ATT&CK

Based on the evidence collected, the following techniques may be relevant:

| Technique                                | Why it may apply                                                                                                                                                                                |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **T1566.001 — Spearphishing Attachment** | The email contained an attachment that multiple security vendors identified as phishing-related or malicious.                                                                                   |
| **T1204.002 — Malicious File**           | Relevant if a user was tricked into opening the attachment. The sandbox showed Word activity, but the available evidence did not establish a real user's actions or a complete execution chain. |

I have not mapped additional techniques because I could not confidently confirm the behaviour required to support them.

## 7. Conclusion

This investigation started with an email that appeared to come from Coinbase support but used a different sender domain. The attached Word document was then identified in VirusTotal, where multiple security vendors flagged it as phishing-related or Trojan-like.

The network analysis also identified a shortened URL, a redirect response and a hostname that had been flagged by several vendors. The sandbox recorded Word and Chrome activity along with several temporary files and shortcut artefacts.

Taken together, these findings support treating the attachment as malicious or highly suspicious.

However, I could not confirm the complete redirect sequence, the purpose of every dropped file or whether a malicious payload successfully executed. I also found no evidence in this investigation that would establish a real endpoint had been compromised.

## 8. Recommended Actions

If this email were found in a real organisation, I would recommend:

* Quarantining the email and attachment to prevent further interaction.
* Searching email and endpoint logs for the sender, attachment hash and specific URL.
* Checking whether any users opened the attachment or accessed the suspicious URL.
* Reviewing endpoint telemetry for related Word, browser or suspicious process activity.
* Blocking confirmed malicious indicators where appropriate, while avoiding unnecessary blocking of shared services.
* Escalating the incident if there were evidence of execution, credential exposure or other suspicious activity.

## Skills Demonstrated

* Email header analysis
* Attachment identification and hashing
* VirusTotal reputation analysis
* URL and DNS investigation
* Sandbox behaviour analysis
* IOC identification and documentation
* Evidence-based conclusions and incident-response recommendations
