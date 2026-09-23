# SuperSecretTip - TryHackMe Technical Walkthrough

## 1. Executive Summary

This assessment documents the complete attack path observed while solving the **SuperSecretTip** TryHackMe room in an authorized lab environment.

The chain begins with reconnaissance of the exposed HTTP service and content discovery. Source-code retrieval then turns the application into a white-box target, revealing a custom XOR routine, an authenticated debug workflow, a localhost-only debug-result endpoint, and an unsafe Jinja rendering path.

The web layer is subsequently converted into code execution. After obtaining an initial shell, scheduled-task behavior and filesystem permissions reveal a writable `.profile` used by a login shell, enabling PATH hijacking and lateral movement. Finally, a root scheduled task invokes `curl` with a configuration file under attacker control, creating a root-context file access primitive.

**Public reporting rule:** challenge flags are intentionally redacted. The screenshots are kept for evidence and chronology, but direct flag text is represented as `FLAG_REDACTED` in this report.

---

## 2. Lab Information

| Field | Value |
|---|---|
| Platform | TryHackMe |
| Room | SuperSecretTip |
| Room URL | https://tryhackme.com/room/supersecrettip |
| Assessment Type | Authorized CTF / Training Lab |
| Primary Focus | Web exploitation + Linux privilege escalation |
| Report Status | Portfolio / public-safe version |
| Flag Handling | Redacted |

---

## 3. Methodology

The investigation followed a source-driven workflow:

1. Enumerate network services.
2. Discover hidden web content.
3. Inspect exposed application source.
4. Reconstruct the required secret.
5. Establish an authenticated debug session.
6. Validate SSTI with a harmless expression.
7. Determine a reliable code-execution path.
8. Obtain an initial shell.
9. Enumerate scheduled tasks and writable files.
10. Abuse the target user's shell initialization path.
11. Inspect the root scheduled task.
12. Demonstrate root-context file access.
13. Preserve evidence while redacting challenge answers.

---

# 4. Phase I - Reconnaissance

## 4.1 Port Scanning

The first step was a full TCP service scan against the lab target.

```bash
nmap -sV -p- -T4 TARGET_IP
```

### Evidence

![Port scan](../docs/assets/01_port_scan.png)

The scan identified two relevant services:

- SSH on port `22/tcp`
- HTTP on port `7777/tcp`, served by Werkzeug/Python

This immediately suggested that the HTTP application was the primary attack surface.

---

## 4.2 Web Content Discovery

I enumerated common directories on the application.

```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt \
     -u http://TARGET_IP:7777/FUZZ \
     -fc 403 -c
```

### Evidence

![Content discovery](../docs/assets/02_content_discovery.png)

Two endpoints were especially important:

- `/cloud`
- `/debug`

The presence of a cloud/download-style route and a debug endpoint indicated that application functionality was likely exposed beyond the main page.

---

# 5. Phase II - Source Discovery and Secret Reconstruction

## 5.1 Recovering Application Source

The `/cloud` functionality was examined through Burp Suite. The request format and backend filtering behavior suggested that filename handling was weak.

The lab evidence showed that the source file could be requested through a crafted download parameter.

### Evidence

![Source retrieval](../docs/assets/03_cloud_source_request.png)

The retrieved code revealed:

- the debug workflow
- a custom `debugpassword` module
- an `ip` helper
- the existence of an additional debug-result path

The important lesson is that source disclosure changes an otherwise black-box exercise into a white-box analysis problem.

---

## 5.2 Obtaining the Encrypted Secret Material

The same download functionality was used to retrieve the encrypted tip file needed by the debug authentication logic.

### Evidence

![Encrypted tip retrieval](../docs/assets/04_secret_tip_download.png)

The application compared a transformed form of the submitted password against stored data.

---

## 5.3 Understanding the XOR Routine

The recovered helper implemented XOR using a fixed byte key.

Conceptually:

```text
ciphertext = plaintext XOR key
plaintext  = ciphertext XOR key
```

Because XOR is self-inverse, the same operation can be used to recover the original input when the key and encrypted bytes are both known.

A small local verification script reproduced the transformation.

### Evidence

![XOR verification](../docs/assets/05_xor_decryption.png)

### Security observation

A custom XOR scheme is not a safe password-protection mechanism when the transformation and key are exposed alongside the ciphertext. This room deliberately demonstrates the value of source-code analysis over brute-force guessing.

---

# 6. Phase III - Debug Authentication and SSTI

## 6.1 Inspecting the Debug Dependency

The source of the IP-checking helper showed that the application trusted a request header to represent the client as localhost.

### Evidence

![Debug source](../docs/assets/06_debug_source_request.png)

This is a classic trust-boundary error: client-controlled HTTP metadata was being used as an authorization signal.

---

## 6.2 Establishing the Authenticated Debug Flow

After reproducing the password transformation and obtaining a valid session, the authenticated debug flow could be reached.

The relevant requests were tested in Burp Repeater so the session context and forwarded IP information could be controlled.

### Evidence

![Debug/SSTI request](../docs/assets/07_ssti_confirmation.png)

![Debug result request](../docs/assets/08_debugresult_cookie.png)

---

## 6.3 Confirming Server-Side Template Injection

The debug parameter was tested with a harmless arithmetic expression.

Example:

```text
{{ 1337 * 1337 }}
```

The application evaluated the expression rather than displaying it as plain input.

### Why this matters

This proves that the application is treating user input as template source. In Jinja/Flask, that can expose Python objects and execution primitives far beyond simple string rendering.

### Evidence

![SSTI confirmation](../docs/assets/07_ssti_confirmation.png)

---

# 7. Phase IV - From SSTI to Command Execution

## 7.1 Python Runtime Introspection

The Python class hierarchy was enumerated to identify a subprocess-capable object.

A representative Jinja expression used during analysis was:

```text
{{"".__class__.__mro__[1].__subclasses__()}}
```

### Evidence

![Class enumeration](../docs/assets/09_ssti_class_enumeration.png)

The returned list was then searched locally to identify the runtime index associated with `subprocess.Popen`.

---

## 7.2 Locating the Runtime Primitive

The HTML-encoded output was copied into a local file and searched for the relevant class name.

```bash
python3 -c "print([i for i, x in enumerate(open('index.txt').read().replace('&lt;','<').replace('&gt;','>').split(',')) if 'subprocess.Popen' in x])"
```

### Evidence

![Popen index](../docs/assets/10_popen_index.png)

The exact index is runtime-dependent and can change between interpreter states, so the important lesson is the **method of enumeration**, not memorizing a static value.

---

## 7.3 Command Execution Validation

The resulting execution path was validated with a benign system command in the lab.

The application returned command output, confirming the transition from SSTI to server-side command execution.

---

## 7.4 Obtaining the Initial Shell

The room filtered certain shell characters. To work around input restrictions inside the training environment, a shell command was encoded and reconstructed on the target.

### Evidence

![Encoded payload preparation](../docs/assets/11_base64_shell.png)

![URL-encoded payload](../docs/assets/12_payload_encoded.png)

The listener was prepared locally:

```bash
nc -lnvp 4445
```

### Evidence

![Shell activity](../docs/assets/13_reverse_shell.png)

At this point, the web compromise had been converted into an operating-system shell under the initial lab user.

**Flag:** `FLAG_REDACTED`

---

# 8. Phase V - Lateral Movement

## 8.1 Scheduled Task Enumeration

After obtaining the initial shell, process activity was monitored to identify recurring jobs.

The monitoring output showed:

- a scheduled job executing Bash as another user
- the use of a login shell
- a separate root-level scheduled task

### Evidence

![Cron observation](../docs/assets/14_cron_observation.png)

The login-shell behavior was important because Bash can process `.profile` during startup.

---

## 8.2 Writable `.profile`

Filesystem permissions were reviewed and the target user's `.profile` was found to be writable from the initial account.

### Evidence

![Writable profile](../docs/assets/15_profile_writable.png)

This created a path for environment manipulation.

---

## 8.3 PATH Hijacking

The lab demonstrated a PATH-search weakness where the scheduled Bash login shell inherited a modified PATH.

A controlled `cat` executable was placed in an attacker-controlled directory and made executable.

```bash
mkdir -p /home/INITIAL_USER/bin
chmod +x /home/INITIAL_USER/bin/cat
```

The target `.profile` was then adjusted so the controlled directory appeared before system directories in `PATH`.

### Evidence

![Controlled executable](../docs/assets/16_fake_cat_payload.png)

When the target cron job executed a command by name, shell lookup selected the attacker-controlled program.

This produced a shell in the context of the second user.

---

# 9. Phase VI - Root-Context `curl` Configuration Abuse

## 9.1 Identifying the Root Scheduled Task

The root job invoked `curl` with the `-K` option against a configuration file.

In curl, `-K`/`--config` tells the program to read additional options from a file.

The critical security condition was that the configuration file could be modified by the lower-privileged user.

---

## 9.2 Demonstrating File Access

The configuration file was modified so that the root-run curl process would read a local file and write the response into a location accessible to the current user.

The technique was demonstrated against the room's root-only material.

### Evidence

![Root curl configuration abuse](../docs/assets/17_root_curl_cron.png)

The resulting artifact was then examined and processed using the same source-assisted cryptographic reasoning used earlier in the room.

---

## 9.3 Final Evidence

### Evidence

![Final terminal artifacts](../docs/assets/18_flags_terminal.png)

The challenge flags are omitted from the public write-up.

**Root flag:** `FLAG_REDACTED`

---

# 10. Complete Attack Chain

```text
Network Enumeration
        |
        v
HTTP 7777 / Werkzeug
        |
        v
Content Discovery
        |
        +--> /cloud
        |
        +--> /debug
        |
        v
Source Code Retrieval
        |
        v
XOR Routine Discovery
        |
        v
Secret Reconstruction
        |
        v
Authenticated Debug Session
        |
        v
SSTI Confirmation
        |
        v
Python Runtime Introspection
        |
        v
Command Execution
        |
        v
Initial Shell
        |
        v
Cron / Process Monitoring
        |
        v
Writable .profile
        |
        v
PATH Hijacking
        |
        v
Second User
        |
        v
Root Cron + curl -K
        |
        v
Root-context File Access
        |
        v
Root Flag (redacted)
```

---

# 11. Key Vulnerabilities

## 11.1 Source-Code Disclosure

**Root cause:** downloadable application resources exposed implementation details.

**Impact:** attackers can enumerate hidden routes, understand authorization assumptions, and recover cryptographic material.

**Remediation:**

- prevent deployment of source files into downloadable paths
- use allowlists for downloadable resources
- validate canonical paths server-side
- separate application source from static/download content

---

## 11.2 Weak XOR-Based Secret Protection

**Root cause:** reversible XOR transformation with recoverable key material.

**Impact:** anyone obtaining both inputs can reconstruct the protected value.

**Remediation:**

- use standard password hashing such as Argon2id, scrypt, or bcrypt
- never store reversible password transformations
- protect keys separately when encryption is actually required

---

## 11.3 SSTI in Flask/Jinja

**Root cause:** untrusted input was inserted into a template string.

**Impact:** template evaluation can expose application internals and, in unsafe configurations, reach OS command execution.

**Remediation:**

- never treat user input as template source
- use `render_template()` with data variables
- apply strict input validation
- reduce template/runtime privileges
- add security testing for template injection

---

## 11.4 Header-Based Localhost Trust

**Root cause:** request metadata was trusted for access control.

**Impact:** remote clients may impersonate trusted local-origin requests when the header is not stripped and set by a trusted reverse proxy.

**Remediation:**

- derive client identity from a trusted network boundary
- strip inbound forwarding headers at the proxy
- configure trusted proxy handling explicitly
- do not use a raw client-controlled header as an authorization decision

---

## 11.5 Writable Shell Initialization

**Root cause:** a lower-privileged user could modify a target user's shell initialization file.

**Impact:** login-shell jobs may execute attacker-controlled PATH entries or other startup behavior.

**Remediation:**

- enforce strict ownership and permissions on `.profile`, `.bashrc`, and related files
- avoid privileged jobs that unnecessarily start login shells
- use absolute command paths in automation

---

## 11.6 Attacker-Controlled `curl` Configuration

**Root cause:** a root scheduled task consumed a configuration file writable by another user.

**Impact:** the privileged curl process inherited attacker-selected options.

**Remediation:**

- make configuration files root-owned and non-writable by unprivileged users
- use fixed arguments instead of user-controlled config files
- run scheduled network tasks under the least privilege required
- use absolute file paths and explicit allowlists

---

# 12. Indicators of Compromise for Detection

For a defender reviewing a similar environment, useful signals include:

- unexpected requests for Python source files
- repeated requests to debug or internal-only endpoints
- suspicious `X-Forwarded-For` values
- Jinja template syntax in query parameters
- process creation involving Python -> shell -> curl chains
- changes to shell startup files
- unexpected executables in user-controlled PATH directories
- root cron jobs reading files from non-root-writable locations
- curl configuration files modified outside normal administrative workflows

---

# 13. Lessons Learned

1. Source code frequently reveals the most valuable part of a web application attack surface.
2. Homegrown reversible transformations should not be confused with password hashing.
3. `render_template_string` is dangerous when the template source is influenced by users.
4. Access-control decisions based on client-controlled headers are fragile.
5. Cron, login shells, and environment configuration can combine into privilege-escalation paths.
6. Any privileged process consuming attacker-writable configuration is a serious trust-boundary issue.
7. Documentation should preserve methodology and evidence without publishing answer strings unnecessarily.

---

# 14. Evidence Index

| # | Screenshot | Purpose |
|---:|---|---|
| 00 | `00_room_banner.png` | Room/banner visual |
| 01 | `01_port_scan.png` | Service enumeration |
| 02 | `02_content_discovery.png` | Hidden endpoint discovery |
| 03 | `03_cloud_source_request.png` | Source retrieval |
| 04 | `04_secret_tip_download.png` | Encrypted tip retrieval |
| 05 | `05_xor_decryption.png` | XOR reconstruction |
| 06 | `06_debug_source_request.png` | IP/debug helper analysis |
| 07 | `07_ssti_confirmation.png` | SSTI confirmation |
| 08 | `08_debugresult_cookie.png` | Authenticated internal endpoint |
| 09 | `09_ssti_class_enumeration.png` | Python runtime enumeration |
| 10 | `10_popen_index.png` | Execution primitive discovery |
| 11 | `11_base64_shell.png` | Encoded command preparation |
| 12 | `12_payload_encoded.png` | URL-encoded payload |
| 13 | `13_reverse_shell.png` | Shell evidence |
| 14 | `14_cron_observation.png` | Scheduled task enumeration |
| 15 | `15_profile_writable.png` | Writable `.profile` |
| 16 | `16_fake_cat_payload.png` | PATH hijacking setup |
| 17 | `17_root_curl_cron.png` | Root curl config abuse |
| 18 | `18_flags_terminal.png` | Final lab artifacts |

---

# 15. Conclusion

SuperSecretTip is a strong example of how several individually understandable weaknesses can become a complete attack chain when combined.

The practical progression was:

**enumeration -> source disclosure -> secret reconstruction -> SSTI -> command execution -> shell -> PATH hijacking -> root cron configuration abuse**

The central lesson is methodological: each escalation step was supported by evidence from the application, its source, process behavior, and filesystem permissions rather than by guessing.

This public version intentionally hides challenge flags and answer strings while preserving the technical reasoning, evidence chronology, and defensive lessons needed for portfolio review.
