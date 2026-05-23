# Privileged Role Assignment Response Playbook

**Playbook ID:** IDP-003  
**Detection:** Unauthorized Privileged Role Assignment  
**MITRE ATT&CK:** T1098.003 - Account Manipulation: Additional Cloud Roles  
**Tactic:** Persistence, Privilege Escalation  
**Severity:** Critical  
**Platform:** Azure AD / Entra ID, Microsoft 365  
**Last Updated:** 2025

---

## Overview

This playbook provides structured response guidance when a privileged role (e.g., Global Administrator, Security Administrator, Privileged Role Administrator) is assigned in Azure AD outside of an approved change window or by an unauthorized identity. Unauthorized role assignments are a top persistence and privilege escalation technique used by adversaries post-compromise.

---

## Detection Source

- **Splunk Alert:** `IDP-003 - Privileged Role Assigned Outside Change Window`
- **Log Sources:** Azure AD Audit Logs (`AuditLogs`), Microsoft 365 Unified Audit Log
- **Trigger Condition:** Role assignment to Global Admin, Security Admin, Exchange Admin, or Privileged Role Admin outside of approved maintenance windows

---

## Triage Checklist

### Step 1 — Validate the Alert

- [ ] Confirm the role assignment event in Azure AD Audit Logs
- [ ] Identify: who assigned the role, to whom, and when
- [ ] Check if an approved change ticket exists for this assignment
- [ ] Verify if the assigning account is authorized to assign roles (PIM or IAM admin)
- [ ] Check if the assigning account was recently compromised or flagged

### Step 2 — Target Account Investigation

- [ ] Identify the account that received the new privileged role
- [ ] Review whether this is an existing user, service principal, or newly created account
- [ ] Check account creation date and recent activity
- [ ] Review sign-in history for the newly privileged account (IP, location, device)
- [ ] Check if account has MFA enrolled
- [ ] Review any group memberships and app assignments

### Step 3 — Activity Review Post-Assignment

- [ ] Review all admin actions performed by the account after the role was assigned
- [ ] Check for new user creation, additional role assignments, or policy modifications
- [ ] Review Conditional Access policy changes
- [ ] Check for new or modified OAuth app registrations or consents
- [ ] Look for any email forwarding rules or mailbox delegation changes
- [ ] Review any PIM (Privileged Identity Management) configuration changes

---

## Containment Actions

### If False Positive (Authorized assignment, no ticket filed):

- [ ] Confirm approval with the account's manager and IAM team
- [ ] Document the exception and remediate process gaps
- [ ] Create a change record retroactively if appropriate
- [ ] Close alert with notes and update detection tuning if needed

### If Unauthorized or Unverified:

- [ ] Immediately remove the unauthorized role assignment:
  ```
  Remove-AzureADDirectoryRoleMember -ObjectId <RoleObjectId> -MemberId <UserObjectId>
  ```
- [ ] Revoke all sessions for the privileged account:
  ```
  Revoke-AzureADUserAllRefreshToken -ObjectId <UserObjectId>
  ```
- [ ] Disable the target account immediately
- [ ] Revoke sessions for the assigning account if also suspicious
- [ ] Enable PIM (Privileged Identity Management) if not already active
- [ ] Preserve audit logs and export for IR review
- [ ] Notify IAM team and Security leadership

---

## Escalation Criteria

Escalate to Tier 2 / IR team if:

- Global Administrator role was assigned (highest privilege level)
- Assignment was made by a compromised or non-human account
- Post-assignment activity includes additional role assignments or policy changes
- Multiple privileged roles assigned in a short window (lateral movement indicator)
- Assignment originates from a foreign or anonymized IP

---

## Post-Incident Actions

- [ ] Document full timeline in ticketing system
- [ ] Review entire role assignment history for the past 30 days
- [ ] Enable or enforce PIM for all privileged roles (Just-In-Time access)
- [ ] Implement alerts for ALL privileged role changes (not just outside change windows)
- [ ] Conduct access review to validate all current privileged role holders
- [ ] Recommend enforcing Privileged Access Workstations (PAW) for admin tasks
- [ ] Update IAM policy and change management procedures if gaps identified

---

## References

- [MITRE ATT&CK T1098.003 - Additional Cloud Roles](https://attack.mitre.org/techniques/T1098/003/)
- [Microsoft: Secure privileged access for hybrid and cloud deployments](https://learn.microsoft.com/en-us/azure/active-directory/roles/security-planning)
- [Microsoft: What is Privileged Identity Management?](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-configure)
