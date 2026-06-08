# Detecting AD Initial Access

**Platform:** TryHackMe
**Date:** 2026-06-07

## Overview

This room focused on detecting the early stages of attacker activity within an Active Directory environment. The emphasis was on identifying authentication anomalies, suspicious account activity, and common indicators that attackers have gained initial access to the domain.

## Concepts Learned

### Initial Access in Active Directory

Initial access refers to the first successful compromise of an account or system within an AD environment. Common techniques include:

* Phishing attacks
* Password spraying
* Credential stuffing
* Exploitation of vulnerable services
* Abuse of weak passwords

---

### Authentication Monitoring

Important authentication-related events:

| Event ID | Description                         |
| -------- | ----------------------------------- |
| 4624     | Successful logon                    |
| 4625     | Failed logon                        |
| 4768     | Kerberos TGT request                |
| 4769     | Kerberos service ticket request     |
| 4771     | Kerberos pre-authentication failure |
| 4776     | NTLM authentication                 |

---

### Password Spraying Detection

Indicators:

* Multiple failed logons across many accounts
* Same source IP targeting different users
* Large numbers of Event ID 4625 entries

Detection focus:

* Failed login volume
* Source IP analysis
* Authentication trends

---

### Kerberos-Based Activity

Kerberos authentication generates:

```text
4768 → TGT Request
4769 → TGS Request
4624 → Successful Logon
```

Monitoring abnormal ticket requests can help identify compromised accounts and suspicious activity.

---

### Account Monitoring

Watch for:

* Newly created accounts
* Privileged account usage
* Unusual login times
* Authentication from unfamiliar systems

Relevant events:

| Event ID | Description              |
| -------- | ------------------------ |
| 4720     | Account created          |
| 4722     | Account enabled          |
| 4724     | Password reset attempted |
| 4740     | Account locked out       |

---

## SOC Analyst Notes

When investigating potential initial access:

* Establish a baseline of normal authentication behavior.
* Investigate spikes in failed logons.
* Correlate authentication events across systems.
* Pay attention to privileged account activity.
* Monitor unusual source hosts and login locations.

---

## Key Takeaways

* Initial access often begins with credential abuse.
* Authentication logs provide critical visibility.
* Password spraying creates recognizable patterns.
* Kerberos and NTLM events are valuable investigation sources.
* Event correlation is more effective than analyzing isolated events.
* Early detection can prevent lateral movement and privilege escalation.

## Personal Notes

This room reinforced the importance of monitoring authentication activity and account behavior to detect attackers during the earliest stages of an Active Directory compromise.
