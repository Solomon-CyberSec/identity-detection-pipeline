# Privileged Role Escalation Simulation

**Simulation ID:** SIM-003  
**Validates Detection:** IDP-003 - Privileged Role Assigned Outside Change Window  
**MITRE ATT&CK:** T1098.003 - Account Manipulation: Additional Cloud Roles  
**Environment:** Lab / Azure AD Test Tenant Only  
**Risk Level:** Medium (assigns admin role to test account — must be reverted)

---

## Objective

Simulate unauthorized privileged role assignment in Azure AD by assigning a high-privilege role to a test user outside of a defined change window. Validate that the IDP-003 Splunk detection fires correctly.

---

## Prerequisites

- [ ] Azure AD test tenant with at least 2 test accounts (one admin-capable, one standard)
- [ ] Splunk instance with Azure AD Audit Logs ingested
- [ ] IDP-003 detection rule loaded in Splunk
- [ ] Change window defined in detection (e.g., Mon-Fri 08:00-18:00)
- [ ] Written authorization from lab environment owner

> **IMPORTANT:** After simulation, immediately remove the assigned role. Do NOT leave privileged roles assigned to test accounts.

---

## Simulation Steps

### Method 1: Azure Portal (Manual)

1. Log in to Azure AD portal as the lab admin account
2. Navigate to: `Azure Active Directory` > `Roles and administrators`
3. Click on `Global Administrator` (or `Security Administrator` for lower risk)
4. Click `+ Add assignments`
5. Search for and select the target test user
6. Click `Add`
7. Note the timestamp of the assignment

### Method 2: PowerShell (Lab Use Only)

```powershell
# LAB ENVIRONMENT ONLY - DO NOT USE IN PRODUCTION
# Assigns Security Administrator role to a test user to trigger detection

$tenantId = "<your-lab-tenant-id>"
Connect-AzureAD -TenantId $tenantId

# Get the role object (use Security Administrator to minimize risk)
$role = Get-AzureADDirectoryRole | Where-Object { $_.DisplayName -eq "Security Administrator" }

# If role not yet activated, activate it first
if (-not $role) {
    $roleTemplate = Get-AzureADDirectoryRoleTemplate | Where-Object { $_.DisplayName -eq "Security Administrator" }
    Enable-AzureADDirectoryRole -RoleTemplateId $roleTemplate.ObjectId
    $role = Get-AzureADDirectoryRole | Where-Object { $_.DisplayName -eq "Security Administrator" }
}

# Get target test user
$user = Get-AzureADUser -Filter "UserPrincipalName eq 'testuser@yourlabdomain.onmicrosoft.com'"

# Assign the role
Add-AzureADDirectoryRoleMember -ObjectId $role.ObjectId -RefObjectId $user.ObjectId

Write-Host "Role assigned. Check Azure AD Audit Logs and Splunk for IDP-003 alert." -ForegroundColor Green
```

---

## Expected Results

### Azure AD Audit Logs:
- Event: `Add member to role`
- `Target` field: test user UPN
- `Modified properties`: role display name
- Timestamp outside defined change window

### Splunk Detection (IDP-003):
- Alert fires for role assignment outside change window
- Fields populated: `actor`, `target_user`, `role_name`, `timestamp`

---

## Validation Checklist

- [ ] Azure AD Audit Logs show role assignment event
- [ ] Assignment was outside defined change window
- [ ] Splunk IDP-003 alert fired with correct details
- [ ] **Role was immediately removed after validation**

---

## Cleanup (REQUIRED)

```powershell
# IMMEDIATELY remove the role assignment after validation
Remove-AzureADDirectoryRoleMember -ObjectId $role.ObjectId -MemberId $user.ObjectId
Write-Host "Role removed successfully." -ForegroundColor Green
```

- [ ] Confirm role removal in Azure AD portal
- [ ] Document simulation results

---

## References

- [MITRE ATT&CK T1098.003](https://attack.mitre.org/techniques/T1098/003/)
- [Microsoft: Assign Azure AD roles to users](https://learn.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal)
