# homelab-cyberdefense

## Overview

This project documents the design, deployment, and security monitoring of a personal cybersecurity homelab.

The main objective is to build a realistic Active Directory environment, simulate common cyberattacks, and understand how to detect and mitigate them.

This homelab is also an opportunity to improve my practical skills in system administration, networking, and defensive cybersecurity.

## Hardware Infrastructure

| Device | Operating System | RAM | Role |
|---|---|---|---|
| Dell OptiPlex 9020 | Debian 12 | 8 GB | Main server |
| Secondary PC | Windows | 4 GB | Active Directory client |

The main server runs continuously and hosts the core infrastructure.

## Technologies

- **Samba Active Directory:** Domain controller, DNS, and identity management.
- **Wazuh:** Security monitoring, log analysis, and threat detection.
- **Sysmon:** Windows event monitoring.
- **Tailscale:** Secure remote access to the infrastructure.
- **Docker:** Containerized applications and services.
- **Ansible:** Infrastructure automation.
- **pfSense:** Network segmentation and firewall management (planned).

## Active Directory Environment

- Domain: `luhoop.lan`
- Domain Controller: `srvads.luhoop.lan`
- Operating System: Debian 12
- Authentication: Kerberos
- Directory Services: Samba AD

## Security Scenarios

### Scenario 1 — Kerberoasting

Simulate a Kerberoasting attack against a deliberately vulnerable service account, analyze Kerberos authentication events, and develop detection mechanisms.

MITRE ATT&CK: T1558.003

### Scenario 2 — Password Spraying

Simulate password spraying against test accounts, analyze authentication failures, and implement detection rules.

MITRE ATT&CK: T1110.003

## Project Status

The homelab is currently under development.

The first phase focuses on deploying and configuring Samba Active Directory while maintaining secure remote access through Tailscale.

Future phases will include endpoint monitoring, attack simulations, detection engineering, and infrastructure automation.

## Disclaimer

All security testing is performed in an isolated, authorized lab environment for educational purposes only.
