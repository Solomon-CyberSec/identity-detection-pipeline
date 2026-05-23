# Token Theft / OAuth Consent Abuse Simulation

**Simulation ID:** SIM-004  
**Validates Detection:** IDP-004 - Suspicious Token Use or OAuth Abuse  
**MITRE ATT&CK:** T1528 - Steal Application Access Token  
**Environment:** Lab / Azure AD Test Tenant Only  
**Risk Level:** Medium (grants OAuth app permissions to test account — must be revoked)

---

## Objective

Simulate an OAuth consent abuse attack by registering a test application with broad permissions and granting consent on behalf of a test user. Validate that suspicious OAuth app activity generates detectable events in Azure AD logs and triggers the IDP-004 detection in Splunk.

---

## Prerequisites

- [ ] Azure AD test tenant with at least 1 test user account
- [ ] Azure AD app registration permissions in the test tenant
- [ ] Splunk instance with Azure AD Audit Logs and Sign-In Logs ingested
- [ ] IDP-004 detection rule loaded in Splunk
- [ ] Written authorization from lab environment owner

> **IMPORTANT:** Revoke all OAuth permissions immediately after simulation. Never grant broad permissions to test apps in a tenant with real data.

---

## Simulation Steps

### Method 1: OAuth Consent Phishing (Manual)

#### Step 1: Register a Test Application

1. Navigate to: `Azure Portal` > `Azure Active Directory` > `App registrations`
2. Click `+ New registration`
3. Name: `SimApp-TokenTheft-Test`
4. Supported account types: `Accounts in this organizational directory only`
5. Click `Register`
6. Note the `Application (client) ID`

#### Step 2: Add API Permissions

1. In the app, go to `API permissions` > `+ Add a permission`
2. Select `Microsoft Graph` > `Delegated permissions`
3. Add: `Mail.Read`, `Files.ReadWrite`, `User.Read`
4. Click `Add permissions` (do NOT click Grant admin consent yet)

#### Step 3: Simulate User Consent

1. Construct an OAuth authorization URL:
   ```
   https://login.microsoftonline.com/<tenant-id>/oauth2/v2.0/authorize?
   client_id=<app-client-id>
   &response_type=code
   &redirect_uri=https://localhost
   &scope=Mail.Read+Files.ReadWrite+User.Read
   &state=12345
   ```
2. Open the URL in a browser while logged in as the test user
3. The user consent dialog will appear — click `Accept`
4. This simulates a user consenting to a potentially malicious OAuth app

### Method 2: PowerShell Simulation (Lab Use Only)

```powershell
# LAB ENVIRONMENT ONLY - DO NOT USE IN PRODUCTION
# Simulates OAuth application token acquisition

$tenantId = "<your-lab-tenant-id>"
$clientId = "<your-test-app-client-id>"  # From app registration above
$scope = "https://graph.microsoft.com/Mail.Read https://graph.microsoft.com/Files.ReadWrite"

# This simulates the token request that would occur after OAuth consent
$tokenUrl = "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token"

Write-Host "Simulating OAuth token acquisition for app: $clientId" -ForegroundColor Yellow
Write-Host "Target scopes: $scope" -ForegroundColor Yellow
Write-Host "Check Azure AD Audit Logs and Splunk for IDP-004 detection." -ForegroundColor Green
```

---

## Expected Results

### Azure AD Audit Logs:
- Event: `Consent to application` with the test app name
- `Target` field: test user UPN
- `Modified properties`: permissions granted (Mail.Read, Files.ReadWrite)
- Application marked as non-verified publisher

### Azure AD Sign-In Logs:
- Non-interactive sign-in from the OAuth application
- `AppId` matching the test app's client ID
- Resource access: Microsoft Graph

### Splunk Detection (IDP-004):
- Alert for OAuth app consent with broad permissions
- Fields populated: `user`, `app_id`, `app_name`, `permissions_granted`, `timestamp`

---

## Validation Checklist

- [ ] Azure AD Audit Logs show `Consent to application` event
- [ ] Permissions show Mail.Read and Files.ReadWrite (broad scope indicators)
- [ ] Splunk IDP-004 alert fired with correct details
- [ ] **OAuth app permissions were immediately revoked after validation**
- [ ] **Test app registration was deleted after validation**

---

## Cleanup (REQUIRED)

1. **Remove user consent:**
   - Azure Portal > `Enterprise Applications` > `SimApp-TokenTheft-Test`
   - Click `Permissions` > `Revoke admin consent` or remove user-level consent

2. **Delete the app registration:**
   - Azure Portal > `App registrations` > `SimApp-TokenTheft-Test`
   - Click `Delete`

```powershell
# PowerShell cleanup
$app = Get-AzureADApplication -Filter "DisplayName eq 'SimApp-TokenTheft-Test'"
Remove-AzureADApplication -ObjectId $app.ObjectId
Write-Host "Test application deleted." -ForegroundColor Green
```

---

## References

- [MITRE ATT&CK T1528](https://attack.mitre.org/techniques/T1528/)
- [Microsoft: OAuth consent phishing](https://learn.microsoft.com/en-us/azure/active-directory/manage-apps/protect-against-consent-phishing)
- [Microsoft: Investigate risky OAuth apps](https://learn.microsoft.com/en-us/defender-cloud-apps/investigate-risky-oauth)
