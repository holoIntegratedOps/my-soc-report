---
title: "Follina (MSDT RCE) Exploitation"
date: "2026-01-05"
description: "Remote Code Execution (Client-side Exploitation)"
Tools: [ "VirusTotal", "OLE dump.py" ]
keywords: [ "Follina", "MSDT RCE" ]
---

## Report Details

| Attribute           | Value                                                                 |
|---------------------|-----------------------------------------------------------------------|
| Incident Report     | Follina Remote Code Execution (RCE) via Malicious Office Document     |
| Report Date         | 2026-01-05                                                            |
| Incident ID         | IR-2026-FOLLINA-001                                                   |
| Severity            | Critical                                                              |
| Status              | Under Analysis / Contained                                            |
| Affected Asset      | Endpoint (Windows workstation – Microsoft Office installed)           |
| Attack Vector       | Malicious document masquerading as `.doc`                             |
| Framework Used      | NIST SP 800-61 Rev. 2                                                 |
| Threat Type         | Remote Code Execution (Client-side Exploitation)                      |

## Incident Summary

On a Friday evening, the SOC team received an alert regarding a newly disclosed and actively exploited Remote Code Execution (RCE) vulnerability, known as Follina, leveraging the Microsoft Diagnostic Tool (MSDT).

A suspicious file named sample.doc was identified. Static analysis revealed that the file was not a genuine Word document, but rather a ZIP-based Office Open XML container, crafted to bypass macro security controls and trigger payload execution via MSDT and Dynamic Data Exchange (DDE) mechanisms.

The sample was flagged as malicious by 46 antivirus vendors, confirming active exploitation in the wild.

## Detection & Analysis (NIST SP 800-61 Rev. 2)

Initial Observations

- File extension spoofing detected (.doc masquerading).
- Hash reputation check confirmed widespread detection.
- Internal inspection revealed XML-based execution logic rather than macro-based payloads.

File Metadata & Hashes

- File Name: sample.doc
- Actual Type: ZIP (Office Open XML)
- SHA-256: 4a24048f81afbe9fb62e7a6a49adbd1faf41f266b5f9feecdceb567aec096784
- AV Detection: 46 vendors flagged as malicious

Key Finding: The XML structure invokes external HTML content, which subsequently executes payloads via MSDT, bypassing macro-based security checks.

## Malware Behavior & MITRE ATT&CK Mapping

Execution – TA0002

- T1064 – Scripting
- T1203 – Exploitation for Client Execution
- T1559 – Inter-Process Communication
- T1559.002 – Dynamic Data Exchange (DDE)

Persistence – TA0003

- T1574 – Hijack Execution Flow
- T1574.002 – DLL Side-Loading

Observed behavior includes termination of msdt.exe to manipulate execution flow.

Privilege Escalation – TA0004

- T1055 – Process Injection
- T1055.002 – Extra Window Memory Injection
- T1574.002 – DLL Side-Loading

Defense Evasion – TA0005

- T1036 – Masquerading
- T1055 / T1055.011 – Process & Extra Window Memory Injection
- T1064 – Scripting
- T1221 – Template Injection
- T1497 – Virtualization / Sandbox Evasion
- T1564 / T1564.011 – Hide Artifacts (Hidden Files & Directories)
- T1574 / T1574.002 – Hijacked Execution Flow (DLL Side-Loading)

Discovery – TA0007

- T1012 – Query Registry
- T1018 – Remote System Discovery
- T1083 – File and Directory Discovery
- T1497 – Virtualization / Sandbox Evasion
- T1518 – Software Discovery

Command and Control – TA0011

- T1071 – Application Layer Protocol
- T1095 – Non-Application Layer Protocol
- T1105 – Ingress Tool Transfer
- T1073 – Encrypted Channel

## Attacker and Victim Information

Victim

- Asset Type: Windows endpoint
- Application: Microsoft Office
- User Interaction: Opening malicious document

Attacker

- Attribution: Unknown (Opportunistic exploitation)
- Infrastructure: External remote servers delivering payloads via HTML/MSDT

## Tools Used (Defensive & Analytical)

Hashing tools (SHA-256)

Antivirus / Reputation engines

Static file analysis tools (ExifTool)

ZIP/OpenXML inspection

MITRE ATT&CK framework

SIEM alerting (initial trigger)

## Event Timeline

| Timestamp           | Event Description                                         |
|---------------------|-----------------------------------------------------------|
| 2026-01-05 02:00    | Suspicious file identified                                |
| 2026-01-05 02:05    | Hash reputation check confirmed as malicious              |
| 2026-01-05 02:10    | File unpacked and XML structure analyzed                  |
| 2026-01-05 02:20    | Follina exploitation confirmed                            |
| 2026-01-05 02:30    | Incident escalated to weekend response team               |

## CIA Impact & Attack Impact

*Overall Impact*: Critical – Full endpoint compromise possible.

| Security Principle | Impact Level | Description                                             |
|--------------------|--------------|---------------------------------------------------------|
| Confidentiality    | High         | Possible data exfiltration via command-and-control (C2) |
| Integrity          | High         | Potential arbitrary code execution                      |
| Availability       | Medium       | Process termination and system instability              |

| Overall Impact | Assessment                                               |
|----------------|----------------------------------------------------------|
| Critical       | Full endpoint compromise possible                        |

## Root Cause Analysis

Lack of patching for MSDT-related vulnerabilities.

Reliance on macro-based detection while XML-based payload bypassed controls.

User exposure to malicious document from external source.

## Eradication & Recovery (NIST SP 800-61 Rev. 2)

Isolate affected endpoint.

Remove malicious files and artifacts.

Disable MSDT protocol handlers.

Apply Microsoft security updates and mitigations.

Reset user credentials if exposure is suspected.

Perform full endpoint reimage if compromise is confirmed.

## Recommended Actions

Disable MSDT URL protocol via Group Policy.

Enforce attachment filtering and file type validation.

Implement EDR rules for msdt.exe abuse.

User awareness training on document-based threats.

Continuous threat intelligence ingestion for zero-day exploits.

## Indicators of Compromise (IOCs)

File Hashes: 4a24048f81afbe9fb62e7a6a49adbd1faf41f266b5f9feecdceb567aec096784

Suspicious Processes: msdt.exe, Office spawning child processes

File Artifacts: XML-based payloads within Office OpenXML containers

## MITRE ATT&CK Summary

This incident maps across Execution, Persistence, Privilege Escalation, Defense Evasion, Discovery, and Command & Control, indicating a full kill-chain capable threat, consistent with real-world Follina exploitation campaigns.
