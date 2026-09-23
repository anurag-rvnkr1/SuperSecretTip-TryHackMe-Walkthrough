# 🛡️ SuperSecretTip — TryHackMe Walkthrough

<div align="center">

![TryHackMe](https://img.shields.io/badge/TryHackMe-SuperSecretTip-red?style=for-the-badge\&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Web%20Exploitation%20%7C%20Privilege%20Escalation-blue?style=for-the-badge)

**A complete professional cybersecurity walkthrough documenting web exploitation, source code analysis, Server-Side Template Injection (SSTI), Remote Code Execution, Linux privilege escalation, cron abuse, and security remediation.**

*Created for educational purposes, portfolio demonstration, and authorized security training.*

</div>

---

## 📖 Overview

**SuperSecretTip** is a technically rich **TryHackMe Capture The Flag (CTF)** room focused on exploiting vulnerabilities in a vulnerable Flask web application and escalating privileges on a Linux machine through multiple misconfigurations.

Rather than relying on brute force or guessing, this room requires **source code analysis**, **understanding application logic**, and chaining several vulnerabilities together to obtain user and root access.

This repository documents the **entire attack path** from reconnaissance to privilege escalation with detailed explanations, screenshots, attack diagrams, mitigation guidance, and GitHub Pages documentation.

> **Portfolio Edition:** All challenge flags, secrets, recovered passwords, cookies, API keys, and sensitive values have been intentionally **redacted** to prevent plagiarism and spoilers while preserving the complete methodology.

---

# 🎯 Objectives

* Enumerate exposed services.
* Discover hidden web application endpoints.
* Analyze exposed application source code.
* Reverse an XOR-based password protection mechanism.
* Exploit **Server-Side Template Injection (SSTI)** in Flask/Jinja2.
* Achieve Remote Code Execution.
* Gain an interactive shell.
* Perform Linux privilege escalation using PATH hijacking.
* Abuse a vulnerable root cron job using `curl -K`.
* Document security findings and defensive mitigations.

---

# 🧩 Skills Demonstrated

<table>
<tr>
<td>

### 🔍 Reconnaissance

* Nmap
* FFUF
* HTTP Enumeration
* Burp Suite Repeater

</td>
<td>

### 🌐 Web Exploitation

* Source Code Disclosure
* Flask Security Analysis
* Jinja2 SSTI
* Request Manipulation
* Header Trust Abuse

</td>
</tr>

<tr>
<td>

### 💻 Linux Privilege Escalation

* Process Enumeration
* Cron Enumeration
* Writable Environment Files
* PATH Hijacking
* File Permission Abuse

</td>
<td>

### 🛡️ Security Concepts

* XOR Cryptography
* Reverse Engineering
* RCE
* Defense-in-Depth
* Secure Configuration Review

</td>
</tr>
</table>

---

# ⚔️ Attack Chain

```text
Network Reconnaissance
        │
        ▼
Port Enumeration
        │
        ▼
Hidden Endpoint Discovery
        │
        ▼
Source Code Disclosure
        │
        ▼
XOR Password Reconstruction
        │
        ▼
Authenticated Debug Session
        │
        ▼
Server-Side Template Injection
        │
        ▼
Remote Code Execution
        │
        ▼
Initial User Shell
        │
        ▼
Cron Enumeration
        │
        ▼
Writable .profile Abuse
        │
        ▼
PATH Hijacking
        │
        ▼
Lateral Movement
        │
        ▼
Root Cron Abuse (curl -K)
        │
        ▼
Root-Level File Access
```

---

# 📚 Walkthrough Sections

The repository documents every stage of the compromise.

| Phase                          | Description                                                            |
| ------------------------------ | ---------------------------------------------------------------------- |
| **1. Reconnaissance**          | Service discovery using Nmap and directory enumeration with FFUF.      |
| **2. Source Code Discovery**   | Downloading and analyzing exposed Flask application source files.      |
| **3. Password Reconstruction** | Understanding and reversing the XOR encryption routine.                |
| **4. Debug Authentication**    | Building a valid debug session and bypassing localhost restrictions.   |
| **5. SSTI Exploitation**       | Confirming template injection and achieving command execution.         |
| **6. Remote Shell Access**     | Executing commands and obtaining an interactive Linux shell.           |
| **7. Privilege Escalation**    | Exploiting writable environment configuration for PATH hijacking.      |
| **8. Root Escalation**         | Abusing a vulnerable root cron configuration using curl.               |
| **9. Security Review**         | Root cause analysis and defensive mitigations for every vulnerability. |

---

# 📂 Repository Structure

```text
SuperSecretTip-TryHackMe-Walkthrough
│
├── README.md
│
├── Documentation
│   ├── Documentation.md
│   └── SuperSecretTip_TryHackMe_Report.docx
│
├── Resources
│   └── notes.md
│
├── docs
│   ├── index.md
│   ├── _config.yml
│   └── assets
│       ├── css
│       │   └── custom.scss
│       ├── 00_room_banner.png
│       ├── 01_port_scan.png
│       ├── 02_content_discovery.png
│       ├── 03_cloud_source_request.png
│       ├── 04_secret_tip_download.png
│       ├── 05_xor_decryption.png
│       ├── 06_debug_source_request.png
│       ├── 07_ssti_confirmation.png
│       ├── 08_debugresult_cookie.png
│       ├── 09_ssti_class_enumeration.png
│       ├── 10_popen_index.png
│       ├── 11_base64_shell.png
│       ├── 12_payload_encoded.png
│       ├── 13_reverse_shell.png
│       ├── 14_cron_observation.png
│       ├── 15_profile_writable.png
│       ├── 16_fake_cat_payload.png
│       ├── 17_root_curl_cron.png
│       └── 18_flags_terminal.png
│
└── LICENSE
```

---

# 📸 Evidence Timeline

Every important stage is supported with screenshots captured during the assessment.

| Screenshot                      | Evidence                                              |
| ------------------------------- | ----------------------------------------------------- |
| `00_room_banner.png`            | TryHackMe room banner.                                |
| `01_port_scan.png`              | Network service discovery.                            |
| `02_content_discovery.png`      | Hidden endpoint enumeration.                          |
| `03_cloud_source_request.png`   | Source code retrieval through `/cloud`.               |
| `04_secret_tip_download.png`    | Encrypted secret retrieval.                           |
| `05_xor_decryption.png`         | XOR password reconstruction.                          |
| `06_debug_source_request.png`   | Debug source analysis.                                |
| `07_ssti_confirmation.png`      | SSTI validation.                                      |
| `08_debugresult_cookie.png`     | Authenticated debug workflow.                         |
| `09_ssti_class_enumeration.png` | Python runtime enumeration.                           |
| `10_popen_index.png`            | Identifying command execution primitive.              |
| `11_base64_shell.png`           | Reverse shell preparation.                            |
| `12_payload_encoded.png`        | Encoded payload generation.                           |
| `13_reverse_shell.png`          | Initial shell access.                                 |
| `14_cron_observation.png`       | Cron monitoring and process analysis.                 |
| `15_profile_writable.png`       | Writable `.profile` discovery.                        |
| `16_fake_cat_payload.png`       | PATH hijacking payload.                               |
| `17_root_curl_cron.png`         | Root cron configuration abuse.                        |
| `18_flags_terminal.png`         | Final privilege escalation evidence (flags redacted). |

---

# 🔐 Vulnerabilities Explored

<table>
<tr>
<th>Vulnerability</th>
<th>Impact</th>
</tr>

<tr>
<td>Source Code Disclosure</td>
<td>Application logic, hidden endpoints, and security mechanisms became visible.</td>
</tr>

<tr>
<td>Weak XOR Secret Storage</td>
<td>Passwords were recoverable because encryption was reversible.</td>
</tr>

<tr>
<td>Server-Side Template Injection (SSTI)</td>
<td>User-controlled input was executed inside the Jinja2 template engine.</td>
</tr>

<tr>
<td>Header Trust Abuse</td>
<td>Localhost authorization relied on attacker-controlled request headers.</td>
</tr>

<tr>
<td>PATH Hijacking</td>
<td>Writable shell initialization enabled execution of attacker-controlled binaries.</td>
</tr>

<tr>
<td>Cron Configuration Abuse</td>
<td>Root scheduled task consumed attacker-controlled curl configuration.</td>
</tr>

</table>

---

# 🛡️ Defensive Security Takeaways

This room demonstrates several real-world secure coding lessons.

### Web Application Security

* Never expose application source files through download endpoints.
* Use proper password hashing algorithms such as **Argon2id**, **bcrypt**, or **scrypt**.
* Never render user-controlled strings using `render_template_string`.
* Validate and sanitize all user-controlled template input.
* Do not trust `X-Forwarded-For` headers unless set by a trusted reverse proxy.

### Linux Security

* Protect shell initialization files (`.profile`, `.bashrc`) with strict ownership.
* Use absolute binary paths inside cron jobs.
* Prevent privileged processes from reading attacker-writable configuration files.
* Audit scheduled tasks for unsafe environment inheritance.

---

# 📑 Documentation Included

| Document                                 | Purpose                              |
| ---------------------------------------- | ------------------------------------ |
| **README.md**                            | Repository landing page.             |
| **Documentation.md**                     | Complete technical walkthrough.      |
| **SuperSecretTip_TryHackMe_Report.docx** | Printable portfolio report.          |
| **Resources/notes.md**                   | Security concepts and key takeaways. |
| **docs/index.md**                        | GitHub Pages documentation site.     |

---

# 🌐 GitHub Pages Documentation

The repository includes a complete **GitHub Pages** site for portfolio presentation.

### Features

* Cybersecurity-themed design.
* Responsive documentation layout.
* Evidence gallery.
* Attack chain explanation.
* Security findings.
* Lessons learned.
* Mitigation guidance.

---

# 🚀 How to View the Documentation

```bash
git clone https://github.com/anurag-rvnkr1/SuperSecretTip-TryHackMe-Walkthrough.git

cd SuperSecretTip-TryHackMe-Walkthrough
```

Open:

```text
docs/index.md
```

or enable **GitHub Pages** using the `/docs` folder.

---

# ⚠️ Flag & Spoiler Policy

To keep this repository educational and portfolio-friendly:

* ✅ Technical methodology is fully documented.
* ✅ Screenshots are preserved as evidence.
* ✅ Exploitation workflow is explained.
* ❌ Challenge flags are replaced with `FLAG_REDACTED`.
* ❌ Sensitive secrets, passwords, cookies, and API keys are removed.

This prevents direct plagiarism while still demonstrating practical offensive security skills.

---

# 🎓 Learning Outcomes

After completing this room, I gained practical experience with:

* Flask application security analysis.
* Source code review during penetration testing.
* XOR cryptography reversal.
* Server-Side Template Injection exploitation.
* Python runtime introspection.
* Linux privilege escalation techniques.
* Scheduled task abuse.
* Secure configuration auditing.
* Vulnerability reporting and documentation.

---

# 📌 Disclaimer

This repository documents an **authorized TryHackMe training environment**.

All exploitation techniques were performed against a deliberately vulnerable lab for educational purposes only.

Do **not** use these techniques against systems without explicit authorization.

---

<div align="center">

### ⭐ If you found this repository useful, consider giving it a Star.

**Cybersecurity Portfolio • TryHackMe Walkthrough Series • By Anurag R.**

</div>
