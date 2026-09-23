---

layout: default
title: SuperSecretTip — TryHackMe Walkthrough
description: Professional penetration testing walkthrough of the SuperSecretTip TryHackMe room by Anurag R.
-----------------------------------------------------------------------------------------------------------

<div align="center">

# 🛡️ SuperSecretTip — TryHackMe Walkthrough

### Professional Cybersecurity Portfolio Documentation

<img src="assets/00_room_banner.png" alt="SuperSecretTip Banner" width="100%">

<br>

![TryHackMe](https://img.shields.io/badge/TryHackMe-SuperSecretTip-red?style=for-the-badge\&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Linux-black?style=for-the-badge\&logo=linux)
![Category](https://img.shields.io/badge/Web-Flask%20%7C%20SSTI%20%7C%20Privilege%20Escalation-blue?style=for-the-badge)

### Author — Anurag R.

*Cybersecurity Portfolio • TryHackMe Walkthrough Series*

</div>

---

## 📖 About This Walkthrough

**SuperSecretTip** is a web exploitation and Linux privilege escalation room on **TryHackMe** that demonstrates how multiple independent security weaknesses can be chained together into a complete system compromise.

This documentation is written as a **professional penetration testing report** rather than a simple CTF write-up. Every exploitation stage is documented with technical analysis, screenshots, security observations, MITRE ATT&CK mapping, and defensive recommendations.

> **Public Portfolio Edition**
>
> All challenge flags, passwords, cookies, API keys, session tokens, and sensitive values have been intentionally **redacted**.

---

# 🎯 Assessment Objectives

* Enumerate exposed services.
* Discover hidden web application endpoints.
* Analyze vulnerable Flask source code.
* Reverse weak XOR authentication logic.
* Authenticate to the internal debug interface.
* Exploit Server-Side Template Injection (SSTI).
* Achieve Remote Code Execution.
* Obtain an interactive Linux shell.
* Escalate privileges using Linux environment misconfigurations.
* Analyze insecure root cron execution.

---

# ⚔️ Complete Attack Chain

```text
Internet
    │
    ▼
Werkzeug HTTP Server
    │
    ▼
Directory Enumeration
    │
    ▼
Source Code Disclosure
    │
    ▼
XOR Secret Reconstruction
    │
    ▼
Authenticated Debug Session
    │
    ▼
Server-Side Template Injection
    │
    ▼
Python Runtime Enumeration
    │
    ▼
Remote Code Execution
    │
    ▼
Interactive Shell
    │
    ▼
Cron Enumeration
    │
    ▼
Writable .profile
    │
    ▼
PATH Hijacking
    │
    ▼
Second User Access
    │
    ▼
Root Cron Abuse
    │
    ▼
Root-Level File Access
```

---

# 🧩 Skills Demonstrated

| Red Team           | Blue Team             | Linux                 |
| ------------------ | --------------------- | --------------------- |
| Nmap Enumeration   | Detection Engineering | Cron Analysis         |
| FFUF Discovery     | MITRE ATT&CK Mapping  | PATH Hijacking        |
| Burp Suite         | Security Hardening    | File Permissions      |
| Source Code Review | Root Cause Analysis   | Environment Variables |
| Flask Security     | Log Analysis          | Privilege Escalation  |
| SSTI Exploitation  | IOC Identification    | Scheduled Tasks       |

---

# 💻 Technical Environment

| Component            | Technology                              |
| -------------------- | --------------------------------------- |
| Operating System     | Ubuntu Linux                            |
| Web Framework        | Flask                                   |
| HTTP Server          | Werkzeug Development Server             |
| Programming Language | Python 3                                |
| Template Engine      | Jinja2                                  |
| Category             | Web Exploitation + Privilege Escalation |

---

# 📍 Walkthrough Timeline

---

## Phase 1 — Network Reconnaissance

### Service Enumeration

<img src="assets/01_port_scan.png" width="100%" alt="Nmap Scan">

A full TCP scan identified:

* SSH Service
* Werkzeug Development Server running on **TCP/7777**

### Key Observation

A development web server exposed to an external network significantly increases the attack surface.

---

## Phase 2 — Web Content Discovery

### Hidden Endpoint Enumeration

<img src="assets/02_content_discovery.png" width="100%" alt="FFUF Enumeration">

Directory enumeration discovered undocumented application endpoints.

### Interesting Endpoints

| Endpoint | Purpose                                         |
| -------- | ----------------------------------------------- |
| `/cloud` | Download functionality exposing internal files. |
| `/debug` | Authentication-protected debug interface.       |

---

## Phase 3 — Source Code Disclosure

### Download Function Analysis

<img src="assets/03_cloud_source_request.png" width="100%" alt="Cloud Endpoint">

The `/cloud` endpoint exposed internal Flask application files due to weak filename validation.

### Security Impact

* Hidden routes exposed.
* Authentication logic exposed.
* Helper modules exposed.
* Cryptographic implementation exposed.

---

### Secret Retrieval

<img src="assets/04_secret_tip_download.png" width="100%" alt="Encrypted Secret">

Encrypted authentication material became accessible through the vulnerable download endpoint.

---

## Phase 4 — XOR Reverse Engineering

<img src="assets/05_xor_decryption.png" width="100%" alt="XOR Reconstruction">

The application relied on a reversible XOR implementation instead of secure password hashing.

### Security Lesson

* Reversible encryption is not password hashing.
* Static XOR keys should never protect authentication secrets.

---

# 🔐 Debug Authentication Analysis

---

## Source Code Review

<img src="assets/06_debug_source_request.png" width="100%" alt="Debug Source">

Application logic revealed how localhost validation and authentication were implemented.

### Trust Boundary Finding

The application trusted client-controlled request metadata when determining localhost access.

---

## Authenticated Debug Session

<img src="assets/08_debugresult_cookie.png" width="100%" alt="Debug Session">

A valid authenticated session was established after reconstructing the protected password.

> **Session cookie intentionally redacted.**

---

# 🌐 Server-Side Template Injection (SSTI)

---

## SSTI Validation

<img src="assets/07_ssti_confirmation.png" width="100%" alt="SSTI Confirmation">

A harmless template expression confirmed Server-Side Template Injection inside the Flask application.

### Why SSTI Matters

SSTI allows attackers to:

* Evaluate server-side expressions.
* Access Python runtime objects.
* Execute operating system commands.
* Escalate to Remote Code Execution.

---

# 🧠 Python Runtime Enumeration

---

## Runtime Class Enumeration

<img src="assets/09_ssti_class_enumeration.png" width="100%" alt="Python Runtime Enumeration">

The Jinja runtime exposed loaded Python classes.

### Enumeration Goal

Identify a runtime object capable of executing operating system commands.

---

## Command Execution Primitive

<img src="assets/10_popen_index.png" width="100%" alt="subprocess Discovery">

The runtime contained a process execution class that enabled command execution.

---

# 💥 Remote Code Execution

---

## Payload Preparation

<img src="assets/11_base64_shell.png" width="100%" alt="Encoded Payload">

The reverse shell command was encoded before delivery.

### Why Encoding?

Encoding bypassed transport restrictions without changing payload functionality.

---

## Payload Delivery

<img src="assets/12_payload_encoded.png" width="100%" alt="URL Encoded Payload">

The encoded payload was delivered through the vulnerable template engine.

---

## Reverse Shell Established

<img src="assets/13_reverse_shell.png" width="100%" alt="Reverse Shell">

The application connected back to the listener, providing an interactive Linux shell.

### Initial Access

```text
FLAG_REDACTED
```

---

# 🐧 Linux Privilege Escalation

---

## Cron Enumeration

<img src="assets/14_cron_observation.png" width="100%" alt="Cron Observation">

Process monitoring revealed recurring scheduled tasks running under multiple user contexts.

### Interesting Discovery

* Login shell executed periodically.
* Root scheduled task executed automatically.

---

## Writable `.profile`

<img src="assets/15_profile_writable.png" width="100%" alt="Writable Profile">

A writable shell initialization file created an opportunity to influence PATH resolution.

### Security Observation

Login shells inherit environment configuration from `.profile`.

---

## PATH Hijacking

<img src="assets/16_fake_cat_payload.png" width="100%" alt="PATH Hijacking">

A malicious executable replaced a legitimate command through PATH precedence.

### Privilege Escalation Technique

* Writable PATH entry.
* Login shell.
* Command executed without absolute path.

---

## Root Cron Configuration Abuse

<img src="assets/17_root_curl_cron.png" width="100%" alt="Root Cron">

A root scheduled task consumed a curl configuration file writable by a lower-privileged user.

### Root Cause

A privileged process trusted attacker-controlled configuration.

---

## Root-Level File Access

<img src="assets/18_flags_terminal.png" width="100%" alt="Root Privilege Escalation">

Root-context file access was achieved through cron configuration abuse.

```text
FLAG_REDACTED
```

---

# 🔎 Security Findings Summary

| Vulnerability                  | Severity    | Category                   |
| ------------------------------ | ----------- | -------------------------- |
| Source Code Disclosure         | 🔴 High     | Information Disclosure     |
| Weak XOR Authentication        | 🟠 Medium   | Cryptographic Weakness     |
| Forwarded Header Trust         | 🔴 High     | Authorization Bypass       |
| Server-Side Template Injection | 🔴 Critical | Remote Code Execution      |
| Writable `.profile`            | 🔴 High     | Privilege Escalation       |
| PATH Hijacking                 | 🔴 High     | Linux Privilege Escalation |
| Writable curl Configuration    | 🔴 Critical | Privilege Escalation       |

---

# 🛡️ Blue Team Detection Opportunities

| Stage                 | Detection Strategy                                         |
| --------------------- | ---------------------------------------------------------- |
| Directory Enumeration | Detect abnormal discovery requests.                        |
| Source Downloads      | Alert on access to application source files.               |
| Debug Interface       | Monitor requests to `/debug`.                              |
| Template Injection    | Detect Jinja syntax inside request parameters.             |
| Runtime Enumeration   | Alert on requests containing Python introspection strings. |
| Reverse Shell         | Detect Python spawning shell processes.                    |
| PATH Hijacking        | Monitor `.profile` modifications.                          |
| Cron Abuse            | Audit privileged scheduled task configuration.             |

---

# 🧬 MITRE ATT&CK Mapping

| ATT&CK Phase                      | Technique |
| --------------------------------- | --------- |
| Network Discovery                 | T1046     |
| File Discovery                    | T1083     |
| Exploit Public-Facing Application | T1190     |
| Command & Scripting Interpreter   | T1059     |
| Process Discovery                 | T1057     |
| Scheduled Task Discovery          | T1053     |
| PATH Interception                 | T1574.007 |
| Data From Local System            | T1005     |

---

# 📚 Security Lessons Learned

## Web Security

* Never expose application source code through download functionality.
* Disable Flask debugging features in production.
* Never trust forwarded headers supplied by clients.
* Never render attacker-controlled template strings.

## Authentication Security

* Replace XOR with Argon2id or bcrypt.
* Separate secrets from application source code.
* Never store reversible authentication secrets.

## Linux Security

* Protect shell initialization files.
* Use absolute executable paths inside cron jobs.
* Prevent privileged tasks from reading writable configuration files.

---

# 🎓 Learning Outcomes

This room strengthened practical understanding of:

### Offensive Security

* Reconnaissance
* Flask Security
* Burp Suite
* Jinja2 SSTI
* Python Runtime Enumeration
* Remote Code Execution
* Linux Privilege Escalation

### Defensive Security

* Secure Configuration Review
* MITRE ATT&CK Mapping
* Detection Engineering
* Root Cause Analysis
* Security Hardening

---

# 📂 Project Structure

```text
SuperSecretTip-TryHackMe-Walkthrough/
│
├── README.md
├── Documentation/
│   ├── Documentation.md
│   └── SuperSecretTip_TryHackMe_Report.docx
├── Resources/
│   └── notes.md
├── docs/
│   ├── index.md
│   ├── _config.yml
│   └── assets/
│       ├── css/
│       └── 00_room_banner.png ... 18_flags_terminal.png
```

---

# 📖 Repository Documentation

| File                                     | Description                            |
| ---------------------------------------- | -------------------------------------- |
| **README.md**                            | Repository landing page.               |
| **Documentation.md**                     | Complete technical walkthrough.        |
| **Resources/notes.md**                   | Security concepts and interview notes. |
| **SuperSecretTip_TryHackMe_Report.docx** | Printable report.                      |
| **docs/index.md**                        | GitHub Pages documentation site.       |

---

# ⚠️ Ethical Disclosure

This documentation was created during an **authorized TryHackMe training exercise**.

### Public Repository Policy

* ✅ Methodology documented.
* ✅ Screenshots preserved.
* ✅ Security analysis included.
* ❌ Challenge flags removed.
* ❌ Passwords removed.
* ❌ Session cookies removed.
* ❌ Secrets removed.

---

<div align="center">

# ⭐ Thank You for Visiting My Portfolio

## SuperSecretTip — TryHackMe Walkthrough

**Cybersecurity Portfolio by Anurag R.**

*Web Exploitation • Linux Privilege Escalation • Security Documentation*

</div>
