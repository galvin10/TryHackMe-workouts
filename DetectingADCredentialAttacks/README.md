# Detecting AD Credential Attacks

**Platform:** TryHackMe
**Date:** 2026-06-10

## Overview

This room focused on detecting common Active Directory credential attacks used by adversaries to gain privileged access. The techniques covered included:

* Kerberoasting
* AS-REP Roasting
* LSASS Credential Dumping
* DCSync

Each technique targets a different stage of credential theft and privilege escalation within Active Directory environments.

---

## Kerberoasting

### What is Kerberoasting?

Kerberoasting allows attackers to request Kerberos service tickets (TGS) for service accounts and crack them offline.

Attack flow:

1. Authenticate as a domain user.
2. Request a TGS ticket for an SPN.
3. Extract the ticket.
4. Crack the service account password offline.

### Why It Works

Common causes:

* Weak service account passwords
* Domain Admin service accounts
* Legacy configurations
* Shared service accounts

### Detection

**Event ID:** 4769

Important fields:

| Field                  | Description              |
| ---------------------- | ------------------------ |
| Service_Name           | Targeted service account |
| Account_Name           | Requesting user          |
| Client_Address         | Source IP                |
| Ticket_Encryption_Type | Encryption used          |

### Encryption Types

| Value | Type     |
| ----- | -------- |
| 0x12  | AES-256  |
| 0x17  | RC4-HMAC |

Detection focus:

* RC4 requests in AES environments
* Multiple service tickets requested
* Same user targeting many SPNs

### Key Takeaway

A burst of Event 4769 entries using RC4 (0x17) from a single account is a strong Kerberoasting indicator.

---

## AS-REP Roasting

### What is AS-REP Roasting?

AS-REP Roasting targets accounts that have Kerberos preauthentication disabled.

Unlike Kerberoasting:

* No valid credentials are required.
* The attacker requests an AS-REP response directly.
* The encrypted response can be cracked offline.

### Detection

**Event ID:** 4768

Suspicious indicators:

* Pre_Authentication_Type = 0
* RC4 encryption
* No follow-up authentication activity

### Investigation Clue

If an account requests a TGT but never generates:

* Event 4624
* Event 4769

the ticket may have been collected solely for offline cracking.

### Key Takeaway

Accounts with preauthentication disabled significantly increase credential theft risk.

---

## LSASS Credential Dumping

### What is LSASS?

LSASS (Local Security Authority Subsystem Service) stores credentials for authenticated users.

It may contain:

* NTLM hashes
* Kerberos tickets
* Cached credentials
* Plaintext passwords (legacy configurations)

### Why Attackers Target LSASS

Dumping LSASS provides immediate access to credentials without requiring password cracking.

Possible post-exploitation techniques:

* Pass-the-Hash
* Pass-the-Ticket
* Lateral Movement

---

### Detection with Sysmon Event 10

**Event ID:** 10 (Process Access)

Important fields:

| Field         | Description             |
| ------------- | ----------------------- |
| SourceImage   | Process accessing LSASS |
| SourceUser    | User executing process  |
| TargetImage   | lsass.exe               |
| GrantedAccess | Requested permissions   |
| CallTrace     | Access method           |

### Common Access Values

| Value    | Meaning             |
| -------- | ------------------- |
| 0x1010   | Memory Read Access  |
| 0x1FFFFF | Full Process Access |

### Suspicious Activity

Examples:

```text
w3wp.exe -> lsass.exe
cmd.exe -> lsass.exe
powershell.exe -> lsass.exe
dllhost.exe -> lsass.exe
```

### CallTrace Analysis

#### MiniDump Method

Often contains:

```text
dbgcore.dll
dbghelp.dll
```

Associated with:

* ProcDump
* comsvcs.dll

#### Injection-Based Access

Often contains:

```text
UNKNOWN(...)
```

Associated with:

* Cobalt Strike
* Meterpreter
* In-memory malware

### Key Takeaway

Processes accessing LSASS with memory-read permissions should always be investigated.

---

## DCSync

### What is DCSync?

DCSync abuses Active Directory replication to retrieve password hashes directly from a Domain Controller.

The attacker impersonates a Domain Controller and requests directory replication data.

### Why It Matters

A successful DCSync attack can expose:

* Domain Admin hashes
* Service account hashes
* User password hashes
* Entire AD credential database

### Replication Rights

Important GUID:

```text
1131f6ad-9c07-11d1-f79f-00c04fc2dcd2
```

DS-Replication-Get-Changes-All

This permission allows password replication.

---

### Detection

**Event ID:** 4662

Important fields:

| Field       | Description                    |
| ----------- | ------------------------------ |
| user        | Account performing replication |
| Access_Mask | Permission used                |
| Properties  | Replication GUID               |
| Logon_ID    | Correlation identifier         |

Suspicious indicators:

* Access_Mask = 0x100
* Replication GUIDs present
* User account not ending in $

### Detection Requirements

For DCSync visibility:

1. Audit Directory Service Access enabled
2. SACL configured on Domain Partition

Without both settings, DCSync activity may not generate logs.

### Key Takeaway

DCSync enables attackers to steal every password hash in the domain without touching the NTDS.dit file.

---

## Credential Attack Comparison

| Technique       | Event ID  | Goal                            |
| --------------- | --------- | ------------------------------- |
| Kerberoasting   | 4769      | Crack service account passwords |
| AS-REP Roasting | 4768      | Crack user passwords            |
| LSASS Dumping   | Sysmon 10 | Steal live credentials          |
| DCSync          | 4662      | Dump domain password hashes     |

---

## SOC Analyst Notes

### High-Value Detection Opportunities

* Event 4769 with RC4 encryption (0x17)
* Event 4768 with PreAuthentication disabled
* Unusual LSASS access activity
* Event 4662 with replication rights
* Multiple service accounts targeted from a single workstation
* Non-admin systems performing replication requests

### Investigation Workflow

1. Identify credential access technique.
2. Determine source user.
3. Determine source host.
4. Identify targeted accounts.
5. Investigate lateral movement.
6. Assess privilege escalation impact.

---

## Key Takeaways

* Kerberoasting and AS-REP Roasting rely on offline password cracking.
* LSASS dumping provides immediate credential access.
* DCSync can expose every password hash in Active Directory.
* Event IDs 4768, 4769, 4662, and Sysmon Event 10 are critical for credential attack detection.
* Correlating authentication events with endpoint telemetry provides better detection coverage.
* Service account hygiene and proper auditing are essential for defending Active Directory environments.

## Personal Notes

This room demonstrated how attackers move from credential access to domain compromise. Understanding Kerberos authentication, LSASS access patterns, and Active Directory replication rights is essential for detecting modern credential theft techniques used by ransomware groups and advanced adversaries.
