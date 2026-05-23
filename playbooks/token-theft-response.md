# Token Theft / OAuth Abuse Response Playbook

**Playbook ID:** IDP-004  
**Detection:** Suspicious OAuth Token Usage / Token Theft Indicators  
**MITRE ATT&CK:** T1528 - Steal Application Access Token  
**Tactic:** Credential Access, Persistence  
**Severity:** Critical  
**Platform:** Azure AD / Entra ID, Microsoft 365, OAuth 2.0  
**Last Updated:** 2025

---

## Overview

This playbook provides structured response guidance for token theft incidents, where an adversary steals OAuth access tokens or refresh tokens to authenticate as a legitimate user without needing their password or MFA. Token theft is a primary technique in modern identity attacks and often bypasses traditional MFA controls. Common vectors include phishing (AiTM), malware, and malicious OAuth app consent.

---

## Detection Source

- **Splunk Alert:** `IDP-004 - Suspicious Token Use or OAuth Abuse`
- **Log Sources:** Azure AD Sign-In Logs (`SigninLogs`), Microsoft 365 Unified Audit Log, Azure AD Audit Logs
- **Trigger Conditions:**
  - Sign-in with no MFA claim but access to sensitive resources
  - Token use from IP inconsistent with MFA registration
  - OAuth application granted broad scopes (Mail.Read, Files.ReadWrite.All)
  - Multiple resource access within seconds from same token (automated access pattern)

---

## Triage Checklist

### Step 1 — Validate the Alert

- [ ] Review the sign-in logs for the user — confirm token-based auth (no interactive MFA)
- [ ] Check the IP address associated with token usage vs. user's normal baseline
- [ ] Identify which application and resource the token was used to access
- [ ] Check if the token was used from a new or unmanaged device
- [ ] Review `authenticationDetails` in Azure AD logs for MFA claim absence
- [ ] Verify if a new OAuth application was recently consented to by the user

### Step 2 — Scope and Impact Assessment

- [ ] Identify all resources accessed using the suspicious token (mail, files, Teams, etc.)
- [ ] Review any data accessed or downloaded via Microsoft Graph API
- [ ] Check for new email forwarding rules or inbox rules created
- [ ] Review OneDrive/SharePoint file access and download activity
- [ ] Check for new OAuth app consents or admin consents granted
- [ ] Identify if token was a refresh token (persistent) vs. access token (short-lived)

### Step 3 — OAuth App Review

- [ ] Identify which OAuth applications have delegated permissions for the user
- [ ] Check for recently consented applications with broad permissions
- [ ] Review if any third-party apps have `Mail.Read`, `Calendars.ReadWrite`, or `Files.ReadWrite.All`
- [ ] Check Enterprise Applications in Azure AD for suspicious new registrations
- [ ] Review admin consent grants made in last 7 days

---

## Containment Actions

### Immediate Token Revocation:

- [ ] Revoke ALL active tokens and sessions immediately:
  ```
  Revoke-AzureADUserAllRefreshToken -ObjectId <UserObjectId>
  ```
- [ ] This will terminate all existing token-based sessions across all apps

### OAuth Application Remediation:

- [ ] Remove suspicious OAuth app consent for the user:
  ```
  # Via Azure Portal: Enterprise Applications > [App Name] > Users and Groups > Remove
  # Or via PowerShell: Remove-AzureADServiceAppRoleAssignment
  ```
- [ ] If admin consent was granted, revoke at the tenant level in Azure AD portal
- [ ] Block the suspicious application in Azure AD Enterprise Applications

### Account Protection:

- [ ] Force password reset for the affected user
- [ ] Force MFA re-registration via verified out-of-band channel
- [ ] Enable Conditional Access policy requiring compliant device for the user
- [ ] Review and enable Continuous Access Evaluation (CAE) if not active
- [ ] Notify user via phone or secondary verified channel

---

## Escalation Criteria

Escalate to Tier 2 / IR team if:

- Token was used to access executive mailboxes or sensitive file shares
- Evidence of data exfiltration via Graph API (bulk email export, file downloads)
- OAuth app spread to multiple users (tenant-wide compromise)
- Admin-level OAuth consent was granted (affects entire organization)
- Token theft is linked to an AiTM (Adversary-in-the-Middle) phishing campaign

---

## Post-Incident Actions

- [ ] Document full timeline and impacted resources in ticketing system
- [ ] Submit malicious OAuth app details to Microsoft (report via Azure portal)
- [ ] Enable OAuth app consent policies (restrict user consent, require admin approval)
- [ ] Deploy Microsoft Defender for Cloud Apps policies to detect OAuth abuse
- [ ] Implement Continuous Access Evaluation (CAE) across the tenant
- [ ] Review and restrict which OAuth apps are permitted in tenant
- [ ] Conduct user awareness training on phishing and OAuth consent attacks
- [ ] Evaluate deploying Token Protection (Conditional Access) in Azure AD

---

## References

- [MITRE ATT&CK T1528 - Steal Application Access Token](https://attack.mitre.org/techniques/T1528/)
- [Microsoft: Investigate risky OAuth applications](https://learn.microsoft.com/en-us/defender-cloud-apps/investigate-risky-oauth)
- [Microsoft: Token theft playbook](https://learn.microsoft.com/en-us/security/operations/token-theft-playbook)
- [Microsoft: Continuous Access Evaluation](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/concept-continuous-access-evaluation)
