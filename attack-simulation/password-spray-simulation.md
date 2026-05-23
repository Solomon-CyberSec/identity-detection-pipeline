# Password Spray Attack Simulation

**Simulation ID:** SIM-001  
**Validates Detection:** IDP-001 - Password Spray Attack  
**MITRE ATT&CK:** T1110.003 - Brute Force: Password Spraying  
**Environment:** Lab / Azure AD Test Tenant Only  
**Risk Level:** Low (lab environment, no real credentials targeted)

---

## Objective

Simulate a password spray attack against Azure AD to generate authentication failure logs and validate that the IDP-001 Splunk detection fires correctly within the expected threshold window.

---

## Prerequisites

- [ ] Azure AD test tenant with at least 10 test user accounts
- [ ] Splunk instance with Azure AD Sign-In Logs ingested
- [ ] IDP-001 detection rule loaded in Splunk
- [ ] Written authorization from lab environment owner
- [ ] Confirm no production accounts will be affected

---

## Simulation Steps

### Method 1: Manual Simulation via Browser

1. Open a browser in incognito/private mode
2. Navigate to `https://login.microsoftonline.com`
3. Attempt to log in with each test user account using an incorrect password
4. Repeat for 15+ accounts within a 10-minute window
5. Use the same source IP for all attempts (to simulate spray behavior)

### Method 2: PowerShell Script (Lab Use Only)

```powershell
# LAB ENVIRONMENT ONLY - DO NOT USE IN PRODUCTION
# Simulates failed authentication attempts against Azure AD

$tenantId = "<your-lab-tenant-id>"
$users = @(
    "testuser1@yourlabdomain.onmicrosoft.com",
    "testuser2@yourlabdomain.onmicrosoft.com",
    "testuser3@yourlabdomain.onmicrosoft.com"
    # Add more test users...
)

$fakePassword = ConvertTo-SecureString "WrongPassword123!" -AsPlainText -Force

foreach ($user in $users) {
    try {
        $cred = New-Object System.Management.Automation.PSCredential($user, $fakePassword)
        Connect-AzureAD -Credential $cred -TenantId $tenantId -ErrorAction Stop
    } catch {
        Write-Host "Failed login for: $user (expected)" -ForegroundColor Yellow
    }
    Start-Sleep -Milliseconds 500
}

Write-Host "Simulation complete. Check Azure AD Sign-In Logs and Splunk." -ForegroundColor Green
```

---

## Expected Results

### Azure AD Sign-In Logs:
- Multiple `50126` error codes (Invalid credentials) across different UPNs
- All attempts from same source IP
- `authenticationRequirement: singleFactorAuthentication` (failed before MFA)

### Splunk Detection (IDP-001):
- Alert should fire when 15+ unique users show failed logins from same IP within 10 minutes
- `sourcetype = azure:aad:signin`
- Fields populated: `src_ip`, `user`, `ResultType`, `ResultDescription`

---

## Validation Checklist

- [ ] Azure AD Sign-In Logs show multiple 50126 failures
- [ ] All failures originate from same IP
- [ ] Splunk IDP-001 alert fired within expected time window
- [ ] Alert contains correct source IP, user count, and timestamp
- [ ] No real user accounts were locked out

---

## Cleanup

- [ ] Verify no test accounts were locked (unlock via Azure AD portal if needed)
- [ ] Clear test data from Splunk notable events if desired
- [ ] Document simulation results for detection validation record

---

## References

- [MITRE ATT&CK T1110.003](https://attack.mitre.org/techniques/T1110/003/)
- [Azure AD Sign-In Error Codes](https://learn.microsoft.com/en-us/azure/active-directory/develop/reference-error-codes)
