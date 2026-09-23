---
layout: default
title: SuperSecretTip — TryHackMe Walkthrough
description: Professional portfolio documentation of the SuperSecretTip TryHackMe room by Anurag R.
---

<div align="center">

# 🛡️ SuperSecretTip — TryHackMe Walkthrough

### Professional Cybersecurity Portfolio Documentation

<img src="assets/00_room_banner.png" width="100%" alt="SuperSecretTip Banner"/>

<br>

![TryHackMe](https://img.shields.io/badge/TryHackMe-SuperSecretTip-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Linux-black?style=for-the-badge&logo=linux)
![Category](https://img.shields.io/badge/Web-SSTI%20%7C%20Privilege%20Escalation-blue?style=for-the-badge)

---

**Author:** **Anurag R.**

*Cybersecurity Portfolio • Offensive Security • Web Exploitation • Linux Privilege Escalation*

</div>

---

# 👋 Welcome

Welcome to my documentation of the **SuperSecretTip** TryHackMe room.

This project demonstrates an end-to-end security assessment of a vulnerable Flask application running on Linux. The walkthrough documents every stage of the engagement — from reconnaissance and source code analysis to Server-Side Template Injection (SSTI), Remote Code Execution (RCE), Linux privilege escalation, cron abuse, and final root compromise.

Unlike a traditional CTF write-up, this documentation is structured as a **professional penetration testing report** designed for recruiters, security engineers, and SOC professionals.

> **Portfolio Safe Edition:** All flags, passwords, cookies, API keys, secrets, and challenge answers have been intentionally **redacted**.

---

# 📚 Documentation Overview

<table>
<tr>
<td width="50%">

## 📖 Technical Walkthrough

A detailed penetration testing report documenting:

- Reconnaissance
- Web Enumeration
- Source Code Analysis
- XOR Reverse Engineering
- Debug Authentication
- SSTI Exploitation
- RCE
- Linux Enumeration
- PATH Hijacking
- Root Privilege Escalation

</td>

<td width="50%">

## 🛡️ Security Review

Includes:

- Root Cause Analysis
- MITRE ATT&CK Mapping
- Detection Opportunities
- Blue Team Notes
- Security Recommendations
- Lessons Learned

</td>
</tr>
</table>

---

# 🎯 Room Objectives

- Enumerate exposed services.
- Discover hidden web endpoints.
- Analyze Flask application source code.
- Reverse weak XOR authentication logic.
- Authenticate to internal debug functionality.
- Exploit Server-Side Template Injection.
- Gain Remote Code Execution.
- Obtain an interactive Linux shell.
- Escalate privileges using Linux misconfigurations.
- Analyze insecure cron job execution.

---

# ⚔️ Complete Attack Chain

```text
Internet
    │
    ▼
Werkzeug HTTP Server (7777)
    │
    ▼
Directory Enumeration
    │
    ▼
/cloud Endpoint
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
Root Cron Configuration Abuse
    │
    ▼
Root-Level File Access
```

---

# 🧩 Skills Demonstrated

<table>
<tr>
<td width="33%">

### 🔍 Reconnaissance

- Nmap
- FFUF
- Burp Suite
- HTTP Enumeration

</td>

<td width="33%">

### 🌐 Web Exploitation

- Flask Analysis
- Source Disclosure
- Jinja2 SSTI
- Request Manipulation
- Python Runtime Enumeration

</td>

<td width="33%">

### 🐧 Linux Privilege Escalation

- Cron Enumeration
- PATH Hijacking
- Writable Environment Files
- Root Cron Abuse
- Secure Configuration Analysis

</td>
</tr>
</table>

---

# 🖥️ Technical Environment

| Component | Technology |
|-----------|------------|
| Operating System | Ubuntu Linux |
| Framework | Flask |
| Web Server | Werkzeug |
| Language | Python 3 |
| Template Engine | Jinja2 |
| Category | Web Exploitation + Privilege Escalation |

---

# 📸 Walkthrough Timeline

---

## Phase 1 — Reconnaissance

### Network Service Discovery

<img src="assets/01_port_scan.png" width="100%" alt="Nmap Scan"/>

The initial attack surface contained:

- SSH
- Werkzeug HTTP Server on TCP/7777

---

### Hidden Endpoint Discovery

<img src="assets/02_content_discovery.png" width="100%" alt="FFUF Enumeration"/>

Directory enumeration identified undocumented functionality including `/cloud` and `/debug`.

---

## Phase 2 — Source Code Disclosure

### Download Function Analysis

<img src="assets/03_cloud_source_request.png" width="100%" alt="Cloud Endpoint"/>

The download endpoint exposed internal Flask source code because of weak filename validation.

---

### Secret Retrieval

<img src="assets/04_secret_tip_download.png" width="100%" alt="Encrypted Secret"/>

The vulnerable download functionality exposed encrypted authentication material.

---

### XOR Password Reconstruction

<img src="assets/05_xor_decryption.png" width="100%" alt="XOR Reconstruction"/>

Application authentication relied on a reversible XOR implementation rather than secure password hashing.

---

## Phase 3 — Debug Authentication & SSTI

### Debug Source Analysis

<img src="assets/06_debug_source_request.png" width="100%" alt="Debug Source"/>

Source review revealed how localhost validation was implemented.

---

### SSTI Confirmation

<img src="assets/07_ssti_confirmation.png" width="100%" alt="SSTI Confirmation"/>

A harmless Jinja expression confirmed Server-Side Template Injection.

---

### Authenticated Debug Session

<img src="assets/08_debugresult_cookie.png" width="100%" alt="Debug Session"/>

The recovered authentication secret established a valid debug session.

---

## Phase 4 — Python Runtime Enumeration

### Python Class Enumeration

<img src="assets/09_ssti_class_enumeration.png" width="100%" alt="Runtime Enumeration"/>

Runtime introspection exposed Python's loaded classes.

---

### Command Execution Primitive

<img src="assets/10_popen_index.png" width="100%" alt="Popen Discovery"/>

The runtime contained a process execution class capable of launching operating system commands.

---

## Phase 5 — Remote Code Execution

### Payload Preparation

<img src="assets/11_base64_shell.png" width="100%" alt="Encoded Payload"/>

Reverse shell payload prepared using safe transport encoding.

---

### URL Encoded Delivery

<img src="assets/12_payload_encoded.png" width="100%" alt="Encoded HTTP Payload"/>

Payload delivered through the vulnerable template engine.

---

### Interactive Shell Access

<img src="assets/13_reverse_shell.png" width="100%" alt="Reverse Shell"/>

Successful Remote Code Execution resulted in an interactive Linux shell.

> **User Flag:** `FLAG_REDACTED`

---

# 🐧 Linux Privilege Escalation

---

## Cron Job Discovery

<img src="assets/14_cron_observation.png" width="100%" alt="Cron Observation"/>

Process monitoring revealed recurring scheduled tasks executed under multiple user contexts.

---

## Writable `.profile`

<img src="assets/15_profile_writable.png" width="100%" alt="Writable Profile"/>

A writable shell initialization file enabled manipulation of the execution environment.

---

## PATH Hijacking

<img src="assets/16_fake_cat_payload.png" width="100%" alt="PATH Hijacking"/>

A controlled executable replaced a legitimate command through PATH precedence.

---

## Root Cron Abuse

<img src="assets/17_root_curl_cron.png" width="100%" alt="Root Cron"/>

A root scheduled task trusted attacker-controlled curl configuration.

---

## Root-Level Access

<img src="assets/18_flags_terminal.png" width="100%" alt="Root Privilege Escalation"/>

Root-level file access was achieved through cron configuration abuse.

> **Root Flag:** `FLAG_REDACTED`

---

# 🔐 Vulnerability Summary

| Vulnerability | Severity | Impact |
|---------------|----------|--------|
| Source Code Disclosure | 🔴 High | Hidden application logic exposed. |
| Weak XOR Authentication | 🟠 Medium | Authentication secret recoverable. |
| Header Trust Abuse | 🔴 High | Authorization boundary bypass. |
| Server-Side Template Injection | 🔴 Critical | Remote Code Execution. |
| Writable `.profile` | 🔴 High | PATH Hijacking. |
| Root Cron Configuration Abuse | 🔴 Critical | Privileged file access. |

---

# 🛡️ Blue Team Detection Opportunities

| Attack Stage | Detection Opportunity |
|--------------|----------------------|
| Directory Enumeration | Monitor abnormal 404/403 request volume. |
| Source Downloads | Alert on requests for source or helper files. |
| Debug Access | Monitor `/debug` endpoint usage. |
| SSTI Payloads | Detect `{{ }}` or `{% %}` patterns in HTTP requests. |
| Runtime Enumeration | Alert on `__class__`, `__mro__`, `subclasses`. |
| Reverse Shell | Detect Python spawning shell processes. |
| PATH Hijacking | File Integrity Monitoring for `.profile`. |
| Cron Abuse | Audit privileged scheduled tasks reading writable files. |

---

# 🧬 MITRE ATT&CK Mapping

| Phase | Technique |
|-------|-----------|
| Reconnaissance | T1046 — Network Service Discovery |
| Source Discovery | T1083 — File Discovery |
| Web Exploitation | T1190 — Exploit Public-Facing Application |
| Command Execution | T1059 — Command & Scripting Interpreter |
| Process Discovery | T1057 |
| Cron Enumeration | T1053 — Scheduled Task |
| PATH Hijacking | T1574.007 |
| Data Collection | T1005 |

---

# 📖 Security Lessons Learned

## Offensive Security

- Source disclosure dramatically increases attacker visibility.
- Weak cryptography is easily reversible.
- Jinja SSTI can escalate directly to Remote Code Execution.
- Linux privilege escalation often relies on configuration weaknesses rather than kernel exploits.

## Defensive Security

- Never expose source code through download functionality.
- Disable Flask debug functionality in production.
- Never trust client-controlled forwarded headers.
- Use Argon2id or bcrypt for password storage.
- Protect `.profile` and scheduled task configuration files.
- Use absolute executable paths inside cron jobs.

---

# 🧠 Learning Outcomes

This room provided hands-on experience with:

<table>
<tr>
<td width="50%">

### Red Team Skills

- Reconnaissance
- Web Enumeration
- Burp Suite
- Flask Security
- SSTI Exploitation
- RCE
- Linux Enumeration
- PATH Hijacking

</td>

<td width="50%">

### Blue Team Skills

- Detection Engineering
- Root Cause Analysis
- MITRE ATT&CK
- Secure Coding
- Linux Hardening
- Monitoring Recommendations

</td>
</tr>
</table>

---

# 📂 Project Structure

```text
SuperSecretTip-TryHackMe-Walkthrough
│
├── README.md
├── Documentation
│   ├── Documentation.md
│   └── SuperSecretTip_TryHackMe_Report.docx
├── Resources
│   └── notes.md
├── docs
│   ├── index.md
│   ├── _config.yml
│   └── assets
│       ├── css
│       │   └── custom.scss
│       ├── 00_room_banner.png
│       ├── 01_port_scan.png
│       ├── ...
│       └── 18_flags_terminal.png
```

---

# 📑 Documentation Included

| Document | Description |
|----------|-------------|
| **README.md** | Repository landing page. |
| **Documentation.md** | Complete penetration testing walkthrough. |
| **Resources/notes.md** | Security notes and interview revision guide. |
| **SuperSecretTip_TryHackMe_Report.docx** | Printable technical report. |
| **docs/index.md** | GitHub Pages portfolio documentation. |

---

# ⚠️ Ethical Notice

This documentation was created while completing an **authorized TryHackMe training lab**.

All exploitation techniques were performed against an intentionally vulnerable environment for educational purposes.

### Public Repository Policy

- ✅ Methodology documented.
- ✅ Screenshots included.
- ✅ Security analysis included.
- ❌ Flags removed.
- ❌ Passwords removed.
- ❌ Session cookies removed.
- ❌ Secrets removed.

---

# 🚀 Explore More

This walkthrough is part of my **Cybersecurity Portfolio**, documenting practical offensive security labs from TryHackMe.

Additional portfolio repositories include Active Directory, web exploitation, Linux privilege escalation, and incident response labs.

---

<div align="center">

## ⭐ Thank You for Visiting My Portfolio

**SuperSecretTip — TryHackMe Walkthrough**

*Cybersecurity Portfolio by **Anurag R.***

</div>
