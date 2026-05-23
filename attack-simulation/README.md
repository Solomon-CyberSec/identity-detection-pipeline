# Attack Simulation Scenarios

This folder contains documented attack simulation scenarios used to validate detections in the `identity-detection-pipeline`. Each scenario maps to one or more Splunk detections and MITRE ATT&CK techniques.

These simulations are designed for use in a **lab environment** (e.g., Azure AD test tenant, isolated cloud lab) and should **never be run against production systems**.

---

## Scenarios

| Scenario | MITRE Technique | Detection Validated | Severity |
|---|---|---|---|
| [Password Spray Simulation](./password-spray-simulation.md) | T1110.003 | IDP-001 | High |
| [Impossible Travel Simulation](./impossible-travel-simulation.md) | T1078 | IDP-002 | High |
| [Privileged Role Escalation](./privileged-role-escalation.md) | T1098.003 | IDP-003 | Critical |
| [Token Theft via OAuth Consent](./token-theft-simulation.md) | T1528 | IDP-004 | Critical |

---

## Lab Environment Requirements

- Azure AD / Entra ID test tenant (free developer tenant via Microsoft 365 Developer Program)
- Splunk test instance with Azure AD log ingestion configured
- Azure AD Sign-In Logs and Audit Logs forwarded to Splunk
- At minimum 2 test user accounts (one standard, one simulated admin)

---

## Important

> All simulations in this folder are for **detection validation and educational purposes only**.  
> Always obtain written authorization before running any simulated attacks.  
> Use isolated lab environments exclusively.
