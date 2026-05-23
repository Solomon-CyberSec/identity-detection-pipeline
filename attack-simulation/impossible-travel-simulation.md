# Impossible Travel Attack Simulation

**Simulation ID:** SIM-002  
**Validates Detection:** IDP-002 - Impossible Travel Detected  
**MITRE ATT&CK:** T1078 - Valid Accounts  
**Environment:** Lab / Azure AD Test Tenant Only  
**Risk Level:** Low (uses valid lab credentials from two controlled locations)

---

## Objective

Simulate an impossible travel scenario by authenticating to Azure AD from two geographically distant IP addresses within a short time window, then validate that the IDP-002 Splunk detection fires correctly.

---

## Prerequisites

- [ ] Azure AD test tenant with at least 1 test user account
- [ ] Access to two different IP addresses in geographically distant locations (e.g., US + EU)
- [ ] A VPN service or cloud VM in a different country for the second authentication
- [ ] Splunk instance with Azure AD Sign-In Logs ingested
- [ ] IDP-002 detection rule loaded in Splunk

---

## Simulation Steps

### Method 1: Manual (Two Devices / VPN)

1. **First login:** Authenticate to Azure AD from your normal lab location (e.g., US)
   - Navigate to `https://portal.azure.com` and log in as test user
   - Note the login timestamp and IP address

2. **Second login (within 30 minutes):**
   - Enable a VPN or use a cloud VM in a geographically distant location (e.g., Europe, Asia)
   - Authenticate to Azure AD with the same test user account
   - Note the second login timestamp and IP address

3. The two logins should be:
   - Same user account
   - Different countries/regions
   - Within 60 minutes of each other
   - Distance that makes physical travel impossible

### Method 2: Using Azure Cloud Shell + VPN

```powershell
# Login 1: From your local machine (US or home region)
# Authenticate to Azure AD from local PowerShell or browser

# Login 2: SSH into a cloud VM in a different region (e.g., Azure VM in West Europe)
# From that VM, authenticate to Azure AD:
Connect-AzureAD -TenantId "<lab-tenant-id>" -Credential $testCred

# Both events will appear in Azure AD Sign-In Logs with different IPs
```

---

## Expected Results

### Azure AD Sign-In Logs:
- Two successful authentications for the same UPN within 60 minutes
- `IPAddress` fields showing two different countries
- `Location` field showing two different cities/countries
- `ResultType: 0` (success) for both events

### Splunk Detection (IDP-002):
- Alert fires when same user authenticates from two countries impossible to travel between
- Fields populated: `user`, `src_ip`, `location`, `time_delta`, `distance_km` (if enriched)

---

## Validation Checklist

- [ ] Azure AD Sign-In Logs show both successful authentications
- [ ] Both logins show different countries in `Location` field
- [ ] Time between both logins is within detection window (< 60 minutes)
- [ ] Splunk IDP-002 alert fired
- [ ] Alert contains correct user, both IPs, and time delta

---

## Cleanup

- [ ] Disconnect VPN or stop cloud VM used for second login
- [ ] Clear test data from Splunk notable events if desired
- [ ] Document simulation results for detection validation record

---

## References

- [MITRE ATT&CK T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- [Microsoft: Risky sign-ins in Azure AD Identity Protection](https://learn.microsoft.com/en-us/azure/active-directory/identity-protection/concept-identity-protection-risks)
