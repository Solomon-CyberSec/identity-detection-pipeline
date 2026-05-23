# IR Playbook: Password Spray Attack

**Detection:** Password Spray Attack
**MITRE Technique:** T1110.003 - Password Spraying
**Severity:** High
**Last Updated:** 2026

---

## Overview

A password spray attack involves an attacker trying a small number of commonly used passwords against many different accounts. Unlike brute force, this technique avoids account lockouts by staying below the lockout threshold per account.

---

## Detection Criteria

- Single source IP attempting authentication against 10+ unique accounts
- Within a 10-minute window
- Failure-to-account ratio < 5 (spread across many accounts)
- Source: Azure AD Sign-in Logs (ResultType != 0)

---

## Triage Steps

### Step 1 - Validate the Alert
- [ ] Review the source IP: Is it known? Internal or external?
- [ ] Check the targeted accounts: Are they related (same department, same naming convention)?
- [ ] Review the time window: Is this during business hours or off-hours?
- [ ] Check for any successful logins from the same IP (ResultType = 0)

### Step 2 - Assess Scope
- [ ] Run SPL to find if any account was successfully compromised:
  ```
  index=azure sourcetype=azure:aad:signin IPAddress="<ATTACKER_IP>" ResultType=0
  ```
- [ ] Check if the IP appears in threat intel feeds
- [ ] Identify how many accounts were targeted vs how many exist

### Step 3 - Determine Intent
- [ ] Is the IP associated with a known security tool or scanner?
- [ ] Is the IP from a known hostile country or ASN?
- [ ] Has this IP appeared in previous alerts?

---

## Containment Actions

### If Attack is Confirmed:
- [ ] Block source IP at the firewall / Conditional Access Named Locations
- [ ] Force password reset for all targeted accounts
- [ ] Enable MFA for any accounts that don't have it
- [ ] Review and enable Conditional Access policies blocking legacy auth

### If Any Account Was Compromised:
- [ ] Immediately revoke all active sessions: `Revoke-AzureADUserAllRefreshToken`
- [ ] Disable the account temporarily
- [ ] Escalate to Incident Response team
- [ ] Preserve logs and initiate full investigation

---

## Escalation Criteria

Escalate to Tier 2 / IR team if:
- Any account shows successful login from the spray IP
- 50+ accounts were targeted
- Attack occurs outside business hours
- IP is from a sanctioned country

---

## Post-Incident Actions

- [ ] Document the incident in ticketing system
- [ ] Submit IP to threat intel sharing platform
- [ ] Review Conditional Access policies for gaps
- [ ] Recommend enabling Password Protection (Azure AD)
- [ ] Update detection thresholds if needed

---

## References

- [MITRE ATT&CK T1110.003](https://attack.mitre.org/techniques/T1110/003/)
- [Microsoft: Password Spray Attacks](https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-password-smart-lockout)
- [Azure AD Sign-in Error Codes](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-aadsts-error-codes)
