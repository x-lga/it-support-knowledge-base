# M365-003 - SharePoint Online: Permission Inheritance Breaks and Unique Permissions

**Article ID:** M365-003
**Category:** Microsoft 365 - SharePoint / OneDrive
**Severity:** P3 (user cannot access a document) | P2 (team cannot access a shared resource)
**Cert alignment:** AZ-900
**Last verified:** 2026-07

---

## The Inheritance Problem That Causes 80% of SharePoint Access Tickets

SharePoint Online permissions cascade from top to bottom by default:
Site → Library → Folder → Document. A user with Read access to the site
inherits Read access to all libraries, folders, and documents in it.

Inheritance breaks happen when:
- Someone clicks "Share" on a specific folder or document (creates unique permissions)
- A workflow or Power Automate changes item permissions automatically
- A site admin removes inheritance to create a restricted area
- A user moves a document between libraries (does not inherit the new library's permissions)

Once inheritance is broken at any level, the item has its own permission list.
The site's permissions no longer apply to it. This is the source of almost every
"I can't access this file but I have access to the site" ticket.

---

## Step 1 - Identify Where Inheritance Is Broken

```powershell
# Using PnP PowerShell (install: Install-Module -Name PnP.PowerShell)
Connect-PnPOnline -Url "https://contoso.sharepoint.com/sites/Finance" `
    -Interactive

# Check if a specific library has unique permissions (inheritance broken)
$Library = Get-PnPList -Identity "Documents"
Write-Host "Library has unique permissions: $(-not $Library.HasUniqueRoleAssignments)"
# True = inheritance broken. False = inherits from site.

# Check a specific folder
$Folder = Get-PnPFolder -Url "/sites/Finance/Documents/2026 Reports"
$FolderItem = Get-PnPListItem -List "Documents" -Id $Folder.ListItemAllFields.Id
Write-Host "Folder has unique permissions: $($FolderItem.HasUniqueRoleAssignments)"

# List the unique permissions on an item (when inheritance is broken)
$Permissions = Get-PnPListItemPermission -List "Documents" -Identity $FolderItem.Id
foreach ($Perm in $Permissions) {
    Write-Host "  $($Perm.Member.Title) — $($Perm.RoleDefinitionBindings.Name -join ', ')"
}
```

---
