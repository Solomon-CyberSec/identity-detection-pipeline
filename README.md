# Identity Detection Pipeline

> End-to-end identity threat detection project — from Azure AD / Entra ID log ingestion through Splunk SIEM detections, incident response playbooks, and attack simulation scenarios.

![Platform](https://img.shields.io/badge/Platform-Splunk%20%7C%20Azure%20AD-blue)
![MITRE](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-red)
![Severity](https://img.shields.io/badge/Focus-Identity%20Security-orange)

---

## Project Overview

Identity is the #1 attack surface in modern enterprise environments. This project simulates a real detection engineering workflow for identity-based threats — covering the full lifecycle from raw log ingestion to alert triage.

**What this project demonstrates:**
- Splunk log ingestion configuration for Azure AD / Entra ID
- SPL-based threat detections mapped to MITRE ATT&CK
- Incident response playbooks for each detection
- Attacker behavior scenarios showing what triggers each alert

---

## Pipeline Architecture

```
Azure AD / Entra ID Logs
        |
        v
  Splunk HEC / Syslog
        |
        v
  Index: azure_identity
        |
        v
  CIM Normalization (Authentication Datamodel)
        |
        v
  SPL Detections (Saved Searches / Alerts)
        |
        v
  Alert Triage --> IR Playbook --> Containment
```

---

## Repository Structure

```
identity-detection-pipeline/
|
|-- README.md
|-- splunk-config/
|   |-- inputs.conf          # Log ingestion from Azure AD
|   |-- transforms.conf      # Field extractions and lookups
|   `-- datamodel.conf       # CIM Authentication datamodel mapping
|
|-- detections/
|   |-- password-spray.spl
|   |-- brute-force-login.spl
|   |-- impossible-travel.spl
|   |-- mfa-bypass-legacy-auth.spl
|   |-- privileged-role-assignment.spl
|   |-- suspicious-token-usage.spl
|   |-- guest-account-abuse.spl
|   `-- oauth-app-permissions.spl
|
|-- playbooks/
|   |-- password-spray-response.md
|   |-- impossible-travel-response.md
|   |-- privileged-role-response.md
|   `-- token-theft-response.md
|
`-- attack-simulation/
    `-- scenarios.md         # Attacker behavior mapped to each detection
```

---

## Detections Included

| Detection | MITRE Tactic | Technique | Severity |
|---|---|---|---|
| Password Spray Attack | Credential Access | T1110.003 | High |
| Brute Force Login | Credential Access | T1110.001 | High |
| Impossible Travel | Initial Access | T1078 | High |
| MFA Bypass via Legacy Auth | Defense Evasion | T1556.006 | High |
| Privileged Role Assignment | Privilege Escalation | T1078.004 | Critical |
| Suspicious Token Usage | Credential Access | T1528 | Critical |
| Guest Account Abuse | Initial Access | T1078.004 | Medium |
| OAuth App High Permissions | Persistence | T1528 | High |

---

## Data Sources

| Source | Splunk Sourcetype | Index |
|---|---|---|
| Azure AD Sign-in Logs | `azure:aad:signin` | `azure` |
| Azure AD Audit Logs | `azure:aad:audit` | `azure` |
| Microsoft 365 Activity | `o365:management:activity` | `o365` |
| Windows Security Events | `wineventlog` | `wineventlog` |

---

## Tools & Environment

- **SIEM:** Splunk Enterprise / Splunk Cloud
- **Identity Platform:** Azure Active Directory / Microsoft Entra ID
- **Framework:** MITRE ATT&CK v14
- **Log Format:** JSON (Azure Monitor / Diagnostic Settings)
- **CIM Version:** Splunk Common Information Model 5.x

---

## About

Built by a cybersecurity engineer with hands-on experience in Splunk SIEM administration, Palo Alto Networks firewall management, and enterprise identity security. This project reflects real detection engineering patterns used in production SOC environments.

**Focus Areas:** Detection Engineering | Identity Security | SIEM Operations | Threat Hunting
