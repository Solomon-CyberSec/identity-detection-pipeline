# Impossible Travel Response Playbook

**Playbook ID:** IDP-002  
**Detection:** Impossible Travel / Anomalous Login Location  
**MITRE ATT&CK:** T1078 - Valid Accounts  
**Tactic:** Initial Access, Defense Evasion  
**Severity:** High  
**Platform:** Azure AD / Entra ID, Microsoft 365  
**Last Updated:** 2025

---

## Overview

This playbook provides structured response guidance for alerts triggered by impossible travel events — where a user authenticates from two geographically distant locations within a timeframe that makes physical travel impossible. This behavior may indicate credential compromise, VPN/proxy usage, or account sharing.

---

## Detection Source

- **Splunk Alert:** `IDP-002 - Impossible Travel Detected`
- **Log Sources:** Azure AD Sign-In Logs (`SigninLogs`), Microsoft 365 Unified Audit Log
- **Trigger Condition:** Same account logs in from two countries within 60 minutes where travel is physically impossible

---

## Triage Checklist

### Step 1 — Validate the Alert

- [ ] Confirm both login events are authentic (not duplicate log entries)
- [ ] Check if the user is known to use a VPN, proxy, or anonymizer service
- [ ] Review if user has a recent travel request or is remote-working internationally
- [ ] Verify both IP addresses using threat intel (VirusTotal, Shodan, AbuseIPDB)
- [ ] Check if either IP is a known Tor exit node or datacenter IP

### Step 2 — User Context Investigation

- [ ] Review user's normal login locations (baseline over past 30 days)
- [ ] Check for MFA registration changes in last 7 days
- [ ] Review recent password reset or recovery activity
- [ ] Confirm user's manager and department for risk context
- [ ] Check if user has privileged roles in Azure AD or M365

### Step 3 — Session & Activity Review

- [ ] Review all sessions active during the alert window
- [ ] Check for email forwarding rules created after the anomalous login
- [ ] Review file access and download activity in SharePoint/OneDrive
- [ ] Check for new OAuth application consents granted
- [ ] Review mailbox delegation and send-as changes
- [ ] Look for any inbox rules that delete or redirect mail

---

## Containment Actions

### If False Positive (User using VPN or traveling legitimately):

- [ ] Document the exception in ticketing system
- [ ] Add user to a temporary VPN/travel watchlist for 30 days
- [ ] Consider tuning detection to exclude known corporate VPN exit IPs
- [ ] Close alert as False Positive with notes

### If Confirmed Compromise or Unverified:

- [ ] Immediately revoke all active sessions:
  ```
  Revoke-AzureADUserAllRefreshToken -ObjectId <UserObjectId>
  ```
- [ ] Force MFA re-registration after verifying user identity out-of-band
- [ ] Disable account temporarily pending investigation
- [ ] Block the anomalous IP(s) in Conditional Access or firewall
- [ ] Preserve all relevant logs before any remediation
- [ ] Notify user via phone or secondary verified channel (not email)

---

## Escalation Criteria

Escalate to Tier 2 / IR team if:

- User denies both logins (full account compromise confirmed)
- Anomalous login is from a sanctioned or high-risk country
- Post-login activity includes data exfiltration indicators
- User account has Global Admin, Security Admin, or privileged roles
- Multiple users show simultaneous impossible travel (potential campaign)

---

## Post-Incident Actions

- [ ] Document full timeline in ticketing system
- [ ] Submit anomalous IPs to threat intel sharing platform
- [ ] Review and update Conditional Access policies (enforce location-based controls)
- [ ] Evaluate Named Locations policy in Azure AD
- [ ] Recommend enforcing phishing-resistant MFA (FIDO2 / Certificate-based)
- [ ] Update detection threshold or location baseline if needed

---

## References

- [MITRE ATT&CK T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- [Microsoft: Investigate risky sign-ins](https://learn.microsoft.com/en-us/azure/active-directory/identity-protection/howto-identity-protection-investigate-risk)
- [Azure AD Conditional Access: Named Locations](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/location-condition)
