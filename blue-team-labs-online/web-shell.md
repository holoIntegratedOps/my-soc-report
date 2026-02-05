---
title: "Network Analysis - Web shell"
date: "2026-02-04"
description: "Lateral Movement"
Tools: [ "Wireshark" ]
keywords: [ "Post-Exploitation Analysis" ]
---

## Report Details

| Attribute           | Value                                                             |
|---------------------|-------------------------------------------------------------------|
| Incident Report     | Network Analysis - Web shell                                      |
| Report Date         | 2026-02-04                                                        |
| Severity            | High                                                              |
| Affected Asset      | Apache/2.4.29 (Ubuntu)                                            |
| Attack Vector       | dbfunctions.php, gobuster, sqlmap                                 |
| Framework Used      | NIST SP 800-61 Rev. 2                                             |
| Threat Type         | Lateral Movement                                                  |

## Executive Summary

An investigation was initiated following a SIEM alert for unauthorized internal network reconnaissance. Analysis of network traffic confirmed that a compromised internal host (10.251.96.4) performed directory brute-forcing and SQL injection attempts against a target web server (10.251.96.5). The threat actor successfully bypassed upload restrictions to deploy a PHP-based web shell ([T1505.003](https://attack.mitre.org/techniques/T1505/003/)), gaining arbitrary command execution (RCE) as evidenced by the execution of the whoami command. The threat has been contained, and the malicious file is slated for removal.

## Detection & Analysis (NIST SP 800-61 Rev. 2)

Internal ip 10.251.96.4 was scanning another internal ip 10.251.96.5 for open port. The attacker used gobuster/3.0.1 to scan vulnerable directory and sqlmap/1.4.7 was used for Credential Brute-forcing.

The attacker access edit profile page, uploaded dbfunctions.php, execute the file and retrieved the directory index of uploaded files, run `id` and `whoami` command.

## Tools Used (Defensive & Analytical)

Wireshark

VirusTotal

## Event Timeline

| Timestamp                  | Event Description                                    |
|----------------------------|------------------------------------------------------|
| 07 Feb 2021 16:34:05 GMT   | gobuster/3.0.1 was used to scan vulnerable directory |
| 07 Feb 2021 16:36:51 GMT   | sqlmap/1.4.7 was used for Credential Brute-forcing   |
| 07 Feb 2021 16:37:34 GMT   | attacker access edit profile page                    |
| 07 Feb 2021 16:40:39 GMT   | the attacker uploaded dbfunctions.php                |
| 07 Feb 2021 16:40:43 GMT   | dbfunctions.php was executed                         |

## CIA Impact & Attack Impact

Overall Impact: Critical – The attacker achieved Remote Code Execution (RCE), providing a foothold for lateral movement and full system takeover.

| Security Principle | Impact Level | Description                                                                                                                                                             |
|--------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Confidentiality    | High         | While no large-scale data breach was observed, the attacker executed commands (whoami) to exfiltrate system metadata and user environment details.                      |
| Integrity          | High         | The attacker successfully bypassed security controls to write a malicious file (dbfunctions.php) and execute unauthorized code on the host.                             |
| Availability       | High         | The server remained operational throughout the incident; however, the risk of a future Ransomware deployment or DoS via the shell remains high until eradicated.        |

## Indicators of Compromise (IOCs)

- **File Hashes**: 87f2e28ab7dcb2e2fa9c282be2ed65ec292725742992602f8080294aac55ce83
- **File Artifacts**: dbfunctions.php

## Eradication & Recovery (NIST SP 800-61 Rev. 2)

Eradication:

- **File Deletion**: Securely delete dbfunctions.php from the /uploads/ directory (or identified path) and scan the filesystem for secondary backdoors or "sticky keys" persistence.
- **Session Termination**: Invalidate all active web sessions and force a password reset for any user accounts associated with the sqlmap brute-force attempts.
**Service Hardening**: Review the web server configuration to disable PHP execution within writeable directories (e.g., using .htaccess or web.config).

Recovery:

- **Integrity Check**: Compare the current web root against a known-good backup to ensure no other source files were tampered with.
- **Log Monitoring**: Implement 24/7 heightened monitoring for the source IP (10.251.96.4) to detect further lateral movement attempts.

## Recommended Actions

- **Input Validation**: Implement strict file-type whitelisting (MIME-type checking) rather than relying on file extensions.
- **Network Segmentation**: Apply Zero Trust principles by restricting internal "east-west" traffic between workstations and servers unless explicitly required for business logic.
- **WAF Implementation**: Deploy a Web Application Firewall (WAF) to detect and block common attack patterns like SQL injection and directory traversal at the edge.
