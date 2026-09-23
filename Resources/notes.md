# SuperSecretTip - Security Notes

## Scope

Target: TryHackMe SuperSecretTip lab only.

## Core Findings

| Stage | Observation | Security implication |
|---|---|---|
| Exposure | Web application on 7777/tcp | Adds an HTTP attack surface |
| Discovery | `/cloud` and `/debug` reachable | Hidden functionality became enumerable |
| Source retrieval | Application source was obtainable | Enables white-box vulnerability analysis |
| Secret handling | XOR used with a static key | Confidentiality depends entirely on key secrecy |
| Debug flow | Localhost trust derived from request metadata | Header spoofing can bypass naive origin checks |
| Template handling | User input reached a Jinja template | SSTI can bridge web input to server-side code |
| Execution | Python runtime objects were reachable | Template compromise escalated to command execution |
| Lateral movement | Writable `.profile` + login shell | PATH hijacking becomes possible |
| Root task | `curl` consumed an attacker-writable config | Root-context file read/write becomes possible |

## Evidence Convention

Screenshots are numbered chronologically. The filename number matches the step order used in `Documentation/Documentation.md`.

Challenge flags are deliberately replaced with:

`FLAG_REDACTED`

Sensitive cookies, exact secrets, and direct answer strings should not be copied into public notes.

## Defensive Takeaways

1. Do not expose application source files in production.
2. Never render untrusted data with `render_template_string`.
3. Treat request headers as untrusted input for access control decisions.
4. Protect user shell initialization files from cross-user writes.
5. Avoid root cron jobs that consume attacker-writable configuration files.
6. Minimize privileges of scheduled jobs and use absolute executable paths.
7. Do not rely on homegrown XOR schemes as a security boundary.
