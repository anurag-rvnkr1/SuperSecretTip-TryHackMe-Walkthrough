# 📝 SuperSecretTip — Security Notes

> Companion notes for the **SuperSecretTip** TryHackMe walkthrough. These notes summarize the security concepts, exploitation techniques, privilege escalation path, and defensive lessons demonstrated in the room.

**Room:** SuperSecretTip (TryHackMe)
**Focus:** Web Exploitation • Source Code Analysis • SSTI • Linux Privilege Escalation

---

# 📚 Table of Contents

* Room Summary
* Attack Chain Overview
* Enumeration Notes
* Source Code Analysis
* XOR Encryption Notes
* Server-Side Template Injection (SSTI)
* Remote Code Execution
* Linux Privilege Escalation
* Cron Job Abuse
* Security Misconfigurations
* Defensive Takeaways
* MITRE ATT&CK Mapping
* Tools Used
* Key Interview Notes

---

# 🎯 Room Summary

SuperSecretTip is a web exploitation room where the attacker gains access through **application source code disclosure**, reconstructs a protected password by reversing a weak XOR cipher, exploits a **Server-Side Template Injection (SSTI)** vulnerability inside a Flask application, gains command execution, and escalates privileges using Linux environment misconfigurations and an insecure root cron job.

Unlike many beginner CTFs, this room requires reading and understanding application source code before exploitation.

---

# ⚔️ Complete Attack Chain

```text
Reconnaissance
      │
      ▼
HTTP Enumeration
      │
      ▼
Hidden Endpoints
      │
      ▼
Source Code Disclosure
      │
      ▼
Weak XOR Secret Recovery
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
Writable .profile
      │
      ▼
PATH Hijacking
      │
      ▼
Lateral Movement
      │
      ▼
Root Cron Configuration Abuse
      │
      ▼
Root-Level File Access
```

---

# 🔍 Enumeration Notes

## Services Identified

| Port         | Service              | Notes                                                             |
| ------------ | -------------------- | ----------------------------------------------------------------- |
| **22/tcp**   | OpenSSH              | Linux remote administration service.                              |
| **7777/tcp** | Werkzeug HTTP Server | Flask development server exposing the vulnerable web application. |

## Interesting Endpoints

| Endpoint       | Purpose                                            |
| -------------- | -------------------------------------------------- |
| `/cloud`       | Download functionality exposing application files. |
| `/debug`       | Debug interface requiring authentication.          |
| `/debugresult` | Internal endpoint rendering debug output.          |

### Enumeration Lessons

* Always enumerate non-standard web ports.
* Use directory brute forcing against development web servers.
* Hidden endpoints often reveal administrative functionality.

---

# 🧠 Source Code Analysis Notes

The exposed Flask source code revealed:

* Hidden routes not visible from the homepage.
* Authentication logic.
* Custom helper modules.
* Password verification workflow.
* Internal debug functionality.

## Security Lesson

Source code disclosure changes a penetration test from **black-box** to **white-box**.

### Why This Matters

Developers frequently expose:

* `source.py`
* `app.py`
* `main.py`
* `config.py`
* `.env`
* Backup files
* Debug files

Always test download functionality for path traversal and weak extension validation.

---

# 🔐 XOR Encryption Notes

## Concept

The application protected the debug password using **XOR encryption**.

### XOR Property

```text
Ciphertext = Plaintext XOR Key
Plaintext  = Ciphertext XOR Key
```

The same operation both encrypts and decrypts data.

### Security Weakness

The application stored:

* encrypted value
* encryption function
* static key

Once both are available, plaintext becomes recoverable.

## Why XOR Is Weak Here

* Reversible.
* Fixed key.
* No salt.
* No hashing.
* No key management.

### Secure Alternatives

| Purpose           | Recommended                 |
| ----------------- | --------------------------- |
| Password Storage  | Argon2id                    |
| Password Storage  | bcrypt                      |
| Password Storage  | scrypt                      |
| Secret Encryption | AES-GCM / ChaCha20-Poly1305 |

---

# 🌐 Server-Side Template Injection (SSTI)

## Definition

SSTI occurs when **user input is interpreted as a server-side template** instead of plain text.

### Framework

* Flask
* Jinja2 Template Engine

### Vulnerable Pattern

Rendering user-controlled strings as templates allows Jinja expressions to execute on the server.

### Typical Indicators

* Mathematical expressions evaluate.
* Template syntax renders output.
* Python objects become accessible.

## Impact

Possible attacker capabilities include:

* Reading server variables.
* Inspecting Python runtime.
* Accessing configuration.
* Executing operating system commands.
* Remote Code Execution.

### Prevention

* Use `render_template()`.
* Never use `render_template_string()` with untrusted input.
* Escape template input.
* Validate user-controlled template content.

---

# 💻 Remote Code Execution Notes

## Execution Flow

SSTI allowed interaction with Python runtime objects.

The exploitation path involved:

1. Runtime object enumeration.
2. Locating a command execution primitive.
3. Executing shell commands.
4. Obtaining an interactive Linux shell.

## Key Concepts

* Python introspection.
* Runtime object hierarchy.
* Process execution.
* Command output retrieval.

### Detection Opportunities

Look for:

* Template expressions in requests.
* Runtime enumeration attempts.
* Unexpected shell spawning.
* Python process launching shell commands.

---

# 🐧 Linux Privilege Escalation Notes

## Initial Shell Enumeration

After shell access:

* Enumerate users.
* Check writable files.
* Monitor running processes.
* Inspect scheduled tasks.

### Useful Enumeration Targets

* Cron jobs.
* SUID binaries.
* Writable scripts.
* Writable configuration files.
* Environment variables.
* PATH.

---

## PATH Hijacking

### Concept

Linux searches executables using the `PATH` environment variable.

If an attacker controls an earlier directory inside `PATH`, malicious binaries may execute before legitimate binaries.

### Requirements

* Writable directory.
* PATH modification.
* Scheduled task or privileged process executing commands without absolute paths.

### Why `.profile` Was Important

A login shell reloads environment configuration, including PATH.

### Prevention

* Use absolute paths.
* Protect `.profile`.
* Avoid writable environment files.

---

# ⏰ Cron Job Abuse

## Cron Observations

Two scheduled tasks were identified:

| User          | Observation                                |
| ------------- | ------------------------------------------ |
| Non-root User | Login shell executed periodically.         |
| Root          | Executed curl with external configuration. |

## Security Issue

Privileged processes trusted attacker-controlled configuration files.

### Dangerous Pattern

* Root executes tool.
* Tool loads writable config.
* Config controls privileged behavior.

### Secure Configuration

* Root-owned configuration.
* Read-only permissions.
* Absolute configuration paths.
* Least privilege.

---

# 🌍 HTTP Header Trust Notes

## Forwarded Header Abuse

The application trusted a client-supplied forwarding header when determining localhost access.

### Risk

Clients can spoof forwarding headers if the application trusts them directly.

### Secure Design

* Reverse proxy sets forwarding headers.
* Application trusts proxy only.
* Strip incoming client forwarding headers.

---

# 🚨 Security Misconfigurations Identified

| Misconfiguration                | Security Risk             |
| ------------------------------- | ------------------------- |
| Source code downloadable        | Information disclosure.   |
| Weak XOR secret storage         | Secret recovery.          |
| Debug endpoint exposed          | Increased attack surface. |
| Client-controlled trust header  | Authorization bypass.     |
| SSTI in Flask                   | Remote Code Execution.    |
| Writable `.profile`             | PATH hijacking.           |
| Root cron reads writable config | Privilege escalation.     |

---

# 🛡️ Defensive Security Takeaways

## Web Application Hardening

* Disable debug endpoints in production.
* Remove development artifacts.
* Validate downloadable files.
* Never expose source code.
* Sanitize user-controlled input.

## Authentication

* Hash passwords.
* Never store reversible secrets.
* Separate secrets from application code.

## Linux Hardening

* Restrict writable shell startup files.
* Use immutable cron configuration.
* Use absolute executable paths.
* Monitor cron integrity.

## Monitoring

Security teams should monitor:

* Unexpected Jinja syntax in HTTP requests.
* Abnormal template rendering errors.
* Shell spawned from web processes.
* Changes to `.profile`.
* Cron configuration modifications.

---

# 🧬 MITRE ATT&CK Mapping

| Phase                | ATT&CK Technique                                  |
| -------------------- | ------------------------------------------------- |
| Service Enumeration  | T1046 — Network Service Discovery                 |
| Source Discovery     | T1083 — File and Directory Discovery              |
| Exploitation         | T1190 — Exploit Public-Facing Application         |
| Command Execution    | T1059 — Command and Scripting Interpreter         |
| Shell Access         | T1105 — Ingress Tool Transfer / Command Execution |
| Privilege Escalation | T1574.007 — PATH Interception                     |
| Scheduled Task Abuse | T1053 — Scheduled Task/Cron                       |
| File Collection      | T1005 — Data from Local System                    |

---

# 🧰 Tools Used

| Tool       | Purpose                                     |
| ---------- | ------------------------------------------- |
| Nmap       | Service discovery.                          |
| FFUF       | Directory enumeration.                      |
| Burp Suite | HTTP interception and request manipulation. |
| Python     | XOR analysis and payload generation.        |
| Netcat     | Reverse shell listener.                     |
| pspy64     | Process and cron monitoring.                |
| Linux CLI  | Enumeration and privilege escalation.       |

---

# 💡 Key Interview Notes

## What is SSTI?

A vulnerability where server-side template engines evaluate attacker-controlled template expressions.

---

## Why is XOR insecure for password storage?

XOR is reversible. Password storage requires one-way hashing with salt and adaptive work factors.

---

## Why is `render_template_string()` dangerous?

It evaluates attacker-controlled template content directly inside the Jinja runtime.

---

## What is PATH Hijacking?

Executing attacker-controlled binaries because a privileged process searches a writable directory before system directories.

---

## Why is trusting `X-Forwarded-For` dangerous?

Clients can spoof the header unless it is rewritten by a trusted reverse proxy.

---

## Why is `curl -K` risky?

It loads runtime options from a configuration file. If that file is writable by an attacker, privileged curl execution can be manipulated.

---

# 📖 Lessons Learned

* Source code disclosure can completely change an attack strategy.
* Weak custom cryptography should never replace industry-standard password hashing.
* Debug functionality frequently introduces high-impact attack surfaces.
* Server-Side Template Injection is one of the most dangerous Flask/Jinja vulnerabilities.
* Environment configuration files can become privilege escalation vectors.
* Scheduled tasks must never consume attacker-controlled configuration files.
* Small security misconfigurations can be chained into full system compromise.

---

# 📌 Portfolio Notes

This repository is maintained as part of my **TryHackMe Walkthrough & Cybersecurity Portfolio**.

**Documentation includes:**

* Professional technical walkthrough.
* Security analysis.
* Numbered screenshots.
* GitHub Pages documentation.
* Printable DOCX report.
* Redacted challenge flags for portfolio-safe publication.

---

<div align="center">

**🛡️ Cybersecurity Portfolio • TryHackMe Walkthrough Series**

*Created by Anurag R.*

</div>
