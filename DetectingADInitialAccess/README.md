# Detecting AD Initial Access

**Platform:** TryHackMe
**Date:** 2026-06-07

## Overview

This room focused on detecting initial access attempts against Active Directory-connected services such as IIS web applications, Exchange OWA, and VPN gateways. Although the attacks target different services, they all ultimately authenticate against Active Directory, making them potential entry points into the domain.

## Key Concepts

### Active Directory as a Central Authentication System

Unlike standalone servers, applications connected to Active Directory share the same authentication source. Compromising one service can provide access to broader domain resources.

Common entry points:

* IIS-hosted applications
* Exchange OWA
* VPN gateways
* SharePoint
* ADFS

---

### IIS Authentication

When users authenticate through an IIS application:

* IIS logs HTTP requests
* Windows Security logs record authentication events
* Domain Controllers validate credentials

Relevant Events:

| Event ID | Description           |
| -------- | --------------------- |
| 4624     | Successful Logon      |
| 4625     | Failed Logon          |
| 4776     | Credential Validation |

---

### IIS Log Fields

Important fields during investigations:

| Field        | Purpose                  |
| ------------ | ------------------------ |
| c-ip         | Client IP Address        |
| cs-uri-stem  | Requested Resource       |
| cs-uri-query | Query Parameters         |
| cs-method    | HTTP Method              |
| sc-status    | HTTP Status Code         |
| User-Agent   | Browser/Tool Information |

---

### Web Shell Detection

Indicators:

* Large numbers of HTTP 404 responses
* Requests to unusual `.aspx` files
* Suspicious query strings
* POST requests to unexpected directories

Common web shell location:

```text
/aspnet_client/
```

Suspicious process activity:

```text
w3wp.exe -> cmd.exe
w3wp.exe -> powershell.exe
```

This is a strong indicator of web shell execution.

---

### Exchange OWA Attacks

Important Exchange paths:

| Path | Purpose                |
| ---- | ---------------------- |
| /owa | Outlook Web Access     |
| /ecp | Exchange Control Panel |

Signs of brute-force activity:

* Multiple POST requests to `/owa/auth.owa`
* Numerous Event ID 4625 failures
* Successful Event ID 4624 after repeated failures

---

### Password Spraying vs Brute Force

#### Password Spraying

* One password
* Many accounts

Example:

```text
user1 : Spring2025!
user2 : Spring2025!
user3 : Spring2025!
```

#### Brute Force

* One account
* Many passwords

Example:

```text
admin : Password1
admin : Password2
admin : Password3
```

Understanding the difference is important during investigations.

---

### VPN Authentication

Most enterprise VPN solutions authenticate users against Active Directory through RADIUS and NPS (Network Policy Server).

Authentication Flow:

```text
VPN Gateway
    ↓
NPS (RADIUS)
    ↓
Active Directory
```

Relevant NPS Events:

| Event ID | Description       |
| -------- | ----------------- |
| 6272     | Access Granted    |
| 6273     | Access Denied     |
| 6274     | Request Discarded |

---

### Important NPS Reason Codes

| Code | Meaning                      |
| ---- | ---------------------------- |
| 16   | Invalid Username or Password |
| 48   | User Not Authorized          |
| 65   | Shared Secret Mismatch       |

Reason Code 16 may indicate credential attacks.

---

## SOC Analyst Notes

### Indicators of Initial Access

* Large numbers of failed authentication attempts
* Authentication outside normal business hours
* Logins from unusual IP addresses
* Web shell deployment
* Successful logins following repeated failures
* Access to administrative interfaces such as `/ecp`
* VPN authentication anomalies

### Correlation Matters

A complete investigation often requires multiple log sources:

* IIS Logs
* Windows Security Logs
* Sysmon Logs
* NPS Logs

A single event rarely tells the full story.

### Useful Logon Types

| Logon Type | Meaning                            |
| ---------- | ---------------------------------- |
| 3          | Network Logon                      |
| 8          | NetworkCleartext (Common with IIS) |

These can help identify the source of authentication activity.

---

## Key Takeaways

* IIS, Exchange, and VPN gateways are common AD initial access targets.
* IIS logs provide visibility that Windows Security logs cannot.
* Web shells often reveal themselves through unusual `.aspx` files and `w3wp.exe` child processes.
* OWA brute-force attacks generate identifiable authentication patterns.
* VPN attacks can be detected using NPS Events 6272 and 6273.
* Successful compromise often appears as multiple failures followed by a successful authentication.
* Correlating application logs with Windows Security events provides the clearest picture of attacker activity.
