# Active Directory PowerShell Setup & Configuration Guide

This guide contains all the PowerShell commands used to deploy, configure, and manage Active Directory in an Azure-hosted Windows Server VM.  
It includes installation, domain creation, OU structure, users, groups, and common help desk tasks.

---

## 🚀 Step 1 — Create the Azure Virtual Machine

1. Go to https://azure.microsoft.com/free and create an account.
2. Sign in at https://portal.azure.com.
3. Search for **Virtual machines** → **Create**.
4. Use the following configuration:
   
| Setting | Value |
|--------|--------|
| Resource Group > Create New | RG-AD-LAB |
| Virtual machine name | VM-AD-DC01 |
| Region | East US |
| Image | Windows Server 2025 Datacenter — Gen2 |
| Size | Standard_DC1s_v2 - 1 vcpu, 4 GiB memory |
| Authentication | Password (Assign your username and password|
| Public inbound ports | Allow RDP (3389) |
| OS Disk | Standard SSD |

5. Click **Review + Create**, then **Create**.

---

## 🛠️ Step 2 — Install Active Directory Domain Services (AD DS)

After connecting via RDP, open **PowerShell as Administrator** and run:

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

### Install Group Policy Management Console

Run:
```powershell
Install-WindowsFeature -Name GPMC
```
---

## 🛠️ Step 3 — Promote the Server to a Domain Controller

Use PowerShell to create the forest and domain:

```powershell
Import-Module ADDSDeployment

Install-ADDSForest `
  -DomainName "lab.local" `
  -DomainNetBiosName "LAB" `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "YourDSRMPassword!" -AsPlainText -Force) `
  -Force:$true
```

The server will restart automatically after promotion.

---

## 🛠️ Step 4 — Build the Organizational Structure

- Create Organizational Units

```powershell
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```

- Create Security Groups

```powershell
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```


- Create User Accounts

Run this entire block together — not line by line.

```powershell
# Step 1 — define password
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

# Step 2 — create users
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

# Step 3 — add users to groups
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```
---

## 🛠️ Step 5 Configure Group Policy (PowerShell Version)

- Create the GPO and link it to the IT OU
```powershell
New-GPO -Name "IT Security Policy" -Comment "Security baseline for IT OU"
```
- Link GPO to IT OU
```powershell
New-GPLink -Name "IT Security Policy" -Target "OU=IT,DC=lab,DC=local"
```
- Configure Password Policy Settings
  These settings correspond to: Computer Config → Policies → Windows Settings → Security → Account Policies → Password Policy
   - Minimum password length = 12
  ```powershell
  Set-GPRegistryValue -Name "IT Security Policy" `
  -Key "HKLM\System\CurrentControlSet\Services\Netlogon\Parameters" `
  -ValueName "MinimumPasswordLength" -Type DWord -Value 12
  ```
   - Password must meet complexity requirements = Enabled
  ```powershell
  Set-GPRegistryValue -Name "IT Security Policy" `
  -Key "HKLM\System\CurrentControlSet\Services\Netlogon\Parameters" `
  -ValueName "RequireStrongKey" -Type DWord -Value 1
  ```

- Configure Security Options
  These settings correspond to: Computer Config → Policies → Windows Settings → Security Settings → Local Policies → Security Options

   - Interactive logon: Machine inactivity limit = 900 seconds
  ```powershell
  Set-GPRegistryValue -Name "IT Security Policy" `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System" `
  -ValueName "InactivityTimeoutSecs" -Type DWord -Value 900
  ```

- Block All Removable Storage
  These settings correspond to: Computer Config → Policies → Administrative Templates → System → Removable Storage Access
   ```powershell
  Set-GPRegistryValue -Name "IT Security Policy" `
  -Key "HKLM\Software\Policies\Microsoft\Windows\RemovableStorageDevices" `
  -ValueName "Deny_All" -Type DWord -Value 1
    ```

## 🛠️ Step 6 Common Help Desk Tasks in AD (PowerShell)


- Unlock a locked account

```powershell
Unlock-ADAccount -Identity "carol.jones"
```

- Disable an account (employee offboarding)

```powershell
Disable-ADAccount -Identity "david.smith"
```
 
- Find all currently disabled accounts
```powershell
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

- Audit and reporting
  - Find accounts that have not logged in for 90 days
```powershell
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} -Properties LastLogonDate | Select-Object Name, LastLogonDate
```
 
 - Check group membership for a specific user
```powershell
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

## ✅ Summary
This guide provides a complete PowerShell-driven workflow for deploying and managing Active Directory in a lab environment.
It demonstrates automation, identity management, and real-world admin tasks used in IT operations.






