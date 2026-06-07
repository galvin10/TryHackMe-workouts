# Monitoring Active Directory

**Platform:** TryHackMe
**Date:** 2026-06-07

## Overview

This room covered the fundamentals of monitoring Active Directory environments, focusing on authentication protocols, important Windows Event IDs, audit policies, and identifying suspicious activity through log analysis.

## Key Protocols

| Protocol      | Port         | Purpose                             |
| ------------- | ------------ | ----------------------------------- |
| Kerberos      | 88           | AD Authentication                   |
| LDAP          | 389/636      | Directory queries and modifications |
| SMB           | 445          | File sharing and administration     |
| RDP           | 3389         | Remote desktop access               |
| NetBIOS/LLMNR | 137,138,5355 | Legacy name resolution              |

## Kerberos Authentication Flow

| Step             | Event ID |
| ---------------- | -------- |
| TGT Request      | 4768     |
| TGS Request      | 4769     |
| Successful Logon | 4624     |

### Failed Authentication

* **4771** → Kerberos pre-authentication failure
* Common causes:

  * Wrong password
  * Typing mistakes
  * Authentication issues

## Kerberos Encryption Types

| Value | Encryption |
| ----- | ---------- |
| 0x12  | AES-256    |
| 0x17  | RC4-HMAC   |

RC4 is typically associated with legacy systems and may warrant investigation in modern environments.

## NTLM Authentication

* **4776** → NTLM credential validation

Common scenarios:

* Accessing shares via IP address
* Legacy applications
* Untrusted domain authentication

## Account Management Events

| Event ID | Description              |
| -------- | ------------------------ |
| 4720     | Account created          |
| 4722     | Account enabled          |
| 4724     | Password reset attempted |
| 4725     | Account disabled         |
| 4740     | Account locked out       |

## Group Policy Monitoring

* **5136** → Directory object modification
* Useful for monitoring GPO changes

Potential attacker actions through GPO abuse:

* Deploy ransomware
* Disable security controls
* Establish persistence

## Notes

* Authentication events are primarily logged on Domain Controllers.
* Computer accounts (ending in `$`) generate a large percentage of Kerberos traffic.
* Proper audit policy configuration is essential for visibility.
* Individual events provide context, but correlated event sequences reveal actual activity.

## Command

```powershell
auditpol /get /category:*
```

Used to verify current audit policy settings on a Domain Controller.

## Takeaways

* Understand Kerberos authentication flow (4768 → 4769 → 4624).
* Monitor failed authentication attempts (4771).
* Track account lifecycle events.
* Monitor GPO modifications using Event ID 5136.
* Correlate events instead of analyzing them individually.

