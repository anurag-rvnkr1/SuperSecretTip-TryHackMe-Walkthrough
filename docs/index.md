---
layout: default
title: SuperSecretTip - TryHackMe Walkthrough
description: Portfolio documentation of the SuperSecretTip TryHackMe room, covering reconnaissance, source disclosure, XOR analysis, SSTI, command execution, PATH hijacking and root cron abuse.
---

# SuperSecretTip - TryHackMe Walkthrough

<div align="center">

## Source-Driven Web Exploitation & Linux Privilege Escalation

**Anurag R. | Cybersecurity Lab Documentation**

[Room](https://tryhackme.com/room/supersecrettip) · [Repository](https://github.com/anurag-rvnkr1/SuperSecretTip-TryHackMe-Walkthrough)

</div>

---

## About This Documentation

This site presents the public-safe documentation of my **SuperSecretTip** TryHackMe lab.

The report focuses on methodology, evidence, vulnerability reasoning, and defensive takeaways. Challenge flags and direct answer strings are intentionally redacted to avoid publishing spoilers or enabling copy-paste submissions.

### Attack-chain summary

```text
Recon
  ↓
7777/tcp - Werkzeug
  ↓
Content discovery
  ↓
Source-code disclosure
  ↓
XOR secret reconstruction
  ↓
Authenticated debug flow
  ↓
SSTI
  ↓
Python runtime command execution
  ↓
Initial shell
  ↓
Cron enumeration
  ↓
Writable .profile
  ↓
PATH hijacking
  ↓
Second-user shell
  ↓
Root cron + curl -K abuse
  ↓
Root-context file access
```

---

## Technical Highlights

### Web Layer
- Port and service enumeration
- Hidden endpoint discovery
- Burp Suite request manipulation
- Source-code disclosure analysis
- Flask/Jinja debugging workflow

### Cryptography
- Custom XOR transformation analysis
- Self-inverse XOR reasoning
- Recovery of application secrets in the lab

### Exploitation
- Server-Side Template Injection
- Python runtime introspection
- Command execution from template context
- Reverse-shell delivery in a controlled lab

### Linux Privilege Escalation
- Cron/process monitoring
- Writable shell initialization file
- PATH hijacking
- Root scheduled task analysis
- `curl -K` configuration-file abuse

---

# Walkthrough

## 01 - Reconnaissance

### Port Scan

![Port scan](assets/01_port_scan.png)

The target exposed SSH and a Python Werkzeug web service on port 7777.

---

## 02 - Content Discovery

![Content discovery](assets/02_content_discovery.png)

Directory enumeration revealed `/cloud` and `/debug`.

---

## 03 - Source Disclosure

![Source retrieval](assets/03_cloud_source_request.png)

The download workflow exposed application source, enabling white-box analysis.

---

## 04 - Secret Material

![Encrypted tip](assets/04_secret_tip_download.png)

The protected tip material was recovered for analysis.

---

## 05 - XOR Reconstruction

![XOR analysis](assets/05_xor_decryption.png)

The application used a reversible XOR transform. The public report omits direct secret values.

---

## 06 - Debug Helper

![Debug source](assets/06_debug_source_request.png)

The debug-related source showed a request-header-based localhost trust decision.

---

## 07-08 - Authenticated Debug Flow

![SSTI confirmation](assets/07_ssti_confirmation.png)

![Debug result](assets/08_debugresult_cookie.png)

A valid session allowed the internal debug result path to be reached.

---

## 09-10 - SSTI to Command Execution

![Class enumeration](assets/09_ssti_class_enumeration.png)

![Popen index](assets/10_popen_index.png)

Python runtime classes were enumerated to identify an execution-capable primitive.

---

## 11-13 - Command Delivery & Shell

![Encoded command](assets/11_base64_shell.png)

![Encoded request](assets/12_payload_encoded.png)

![Shell evidence](assets/13_reverse_shell.png)

The web vulnerability was converted into command execution and a shell in the lab.

**Flag: `FLAG_REDACTED`**

---

## 14-16 - Lateral Movement

![Cron observation](assets/14_cron_observation.png)

![Writable profile](assets/15_profile_writable.png)

![Controlled executable](assets/16_fake_cat_payload.png)

A scheduled login shell consumed a writable `.profile`, enabling PATH hijacking and movement to the second user.

---

## 17-18 - Root-Context Access

![Curl configuration abuse](assets/17_root_curl_cron.png)

![Final artifacts](assets/18_flags_terminal.png)

A root scheduled task consumed an attacker-controlled curl configuration file. The final challenge answer is omitted.

**Root flag: `FLAG_REDACTED`**

---

# Security Analysis

## Finding 1 - Exposed Source Files

Application source was accessible through a download function.

**Risk:** implementation details, hidden routes, and secret-handling logic become available to remote users.

**Mitigation:** strict server-side allowlisting and separation of source code from downloadable content.

## Finding 2 - Unsafe Template Rendering

User-controlled input reached Jinja template evaluation.

**Risk:** SSTI may expose internal objects and lead to code execution.

**Mitigation:** render static templates with data variables; never treat user input as template source.

## Finding 3 - Header-Based Trust

A forwarding header influenced the localhost decision.

**Risk:** client-controlled request metadata can cross a trust boundary.

**Mitigation:** set and strip forwarding headers only at trusted proxy boundaries.

## Finding 4 - Writable Shell Initialization

A target `.profile` was writable by a lower-privileged user.

**Risk:** login-shell startup can execute attacker-controlled environment or PATH modifications.

**Mitigation:** strict file ownership, permissions, and avoidance of unnecessary login shells for automation.

## Finding 5 - Privileged curl Configuration

A root task consumed a config file writable by an unprivileged user.

**Risk:** privileged processes inherit attacker-selected command options.

**Mitigation:** protect configuration files, use fixed arguments, absolute paths, and least privilege.

---

# Defensive Lessons

> **The strongest lesson from this room is trust-boundary management.**

The attack chain required several weak assumptions to line up:

- downloadable application internals
- reversible secret handling
- unsafe template rendering
- weak localhost trust
- writable shell initialization
- privileged use of a user-controlled config file

Breaking any one of these links can reduce the overall attack path dramatically.

---

# Evidence Gallery

| Step | Evidence |
|---:|---|
| 01 | [Port scan](assets/01_port_scan.png) |
| 02 | [Content discovery](assets/02_content_discovery.png) |
| 03 | [Source request](assets/03_cloud_source_request.png) |
| 04 | [Secret material](assets/04_secret_tip_download.png) |
| 05 | [XOR analysis](assets/05_xor_decryption.png) |
| 06 | [Debug helper](assets/06_debug_source_request.png) |
| 07 | [SSTI confirmation](assets/07_ssti_confirmation.png) |
| 08 | [Debug result](assets/08_debugresult_cookie.png) |
| 09 | [Class enumeration](assets/09_ssti_class_enumeration.png) |
| 10 | [Popen discovery](assets/10_popen_index.png) |
| 11 | [Encoded shell](assets/11_base64_shell.png) |
| 12 | [Encoded payload](assets/12_payload_encoded.png) |
| 13 | [Shell evidence](assets/13_reverse_shell.png) |
| 14 | [Cron analysis](assets/14_cron_observation.png) |
| 15 | [Writable profile](assets/15_profile_writable.png) |
| 16 | [PATH hijack](assets/16_fake_cat_payload.png) |
| 17 | [Root curl abuse](assets/17_root_curl_cron.png) |
| 18 | [Final artifacts](assets/18_flags_terminal.png) |

---

# Portfolio Notes

**Room:** SuperSecretTip  
**Platform:** TryHackMe  
**Focus:** Web exploitation, source-code analysis, SSTI, RCE, Linux privilege escalation  
**Public flags:** Redacted  
**Use:** Authorized training and portfolio documentation

---

[Back to Repository](https://github.com/anurag-rvnkr1/SuperSecretTip-TryHackMe-Walkthrough)
