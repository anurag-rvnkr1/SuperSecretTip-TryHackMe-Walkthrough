# SuperSecretTip - TryHackMe Walkthrough

> A portfolio-focused, source-driven security assessment of the **SuperSecretTip** room, documenting the complete attack chain from reconnaissance to root-level impact.

[![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=flat-square&logo=tryhackme)](https://tryhackme.com/)
[![Focus](https://img.shields.io/badge/Focus-Web%20%7C%20SSTI%20%7C%20RCE%20%7C%20Linux-blue?style=flat-square)](https://github.com/anurag-rvnkr1/SuperSecretTip-TryHackMe-Walkthrough)
[![Documentation](https://img.shields.io/badge/Docs-GitHub%20Pages-24292f?style=flat-square&logo=github)](https://anurag-rvnkr1.github.io/SuperSecretTip-TryHackMe-Walkthrough/)

## Overview

This repository contains my documented walkthrough of the TryHackMe **SuperSecretTip** room.

The engagement demonstrates a realistic, source-assisted attack path involving:

- service and content discovery
- exposed source-code retrieval
- XOR-based secret reconstruction
- Flask/Jinja Server-Side Template Injection (SSTI)
- command execution and shell access
- PATH hijacking through a writable `.profile`
- scheduled-task abuse
- `curl -K` configuration abuse for root-context file access

The documentation is intentionally written for **authorized lab use, learning, and portfolio review**.

> **Flag policy:** challenge flags and other direct answer strings are redacted from the public documentation to reduce accidental spoilers and plagiarism. Screenshots are retained as visual evidence, with sensitive values omitted where practical.

## Attack Chain

```text
Recon
  |
  +--> Port 7777 / Werkzeug
  |
  +--> Content discovery (/cloud, /debug)
  |
  +--> Source retrieval
  |
  +--> XOR secret reconstruction
  |
  +--> Authenticated debug flow
  |
  +--> SSTI
  |
  +--> Python runtime abuse -> command execution
  |
  +--> Shell as initial user
  |
  +--> Cron inspection
  |
  +--> Writable target .profile
  |
  +--> PATH hijacking -> lateral movement
  |
  +--> Root cron + curl configuration abuse
  |
  +--> Root-context file access
```

## Documentation

The full technical report is available here:

- [Complete CTF Documentation](Documentation/Documentation.md)
- [GitHub Pages Documentation](docs/index.md)
- [Security Notes](Resources/notes.md)

## Repository Layout

```text
SuperSecretTip-TryHackMe-Walkthrough/
├── README.md
├── Documentation/
│   └── Documentation.md
├── Resources/
│   └── notes.md
├── docs/
│   ├── index.md
│   └── assets/
│       ├── css/
│       │   └── custom.scss
│       └── 00_room_banner.png
│       └── 01_port_scan.png
│       └── 02_content_discovery.png
│       └── ...
│       └── 18_flags_terminal.png
└── .gitignore
```

## Learning Outcomes

This lab reinforces practical skills in:

- Nmap and web content enumeration
- Burp Suite request manipulation
- source-code assisted vulnerability analysis
- weak custom cryptography analysis
- Jinja2/Flask SSTI identification
- Python runtime introspection
- command execution through unsafe template rendering
- Linux scheduled-task enumeration
- environment/PATH security
- permissions review
- command-line configuration-file abuse
- documenting an end-to-end attack chain

## Disclaimer

This material describes activity performed against an intentionally vulnerable TryHackMe environment. Do not reproduce these techniques against systems you do not own or have explicit permission to assess.

## Author

**Anurag R.**  
Cybersecurity learner | CTF documentation | Security tooling

Repository: `anurag-rvnkr1/SuperSecretTip-TryHackMe-Walkthrough`
