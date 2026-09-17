# Windows Server 2025 - DC01

> **Lab only:** The following configuration intentionally introduces insecure Active Directory permissions for an isolated penetration-testing environment.

---

## 1. Rename Server

Rename the Windows Server host to `DC01`:

```powershell
Rename-Computer -NewName "DC01" -Restart
```

---

## 2. Configure Static IP

Check the network adapter name:

```powershell
Get-NetAdapter
```

Configure the static IP address:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "10.0.0.10" -PrefixLength 24
```

Configure the local DNS server:

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "127.0.0.1"
```

---

## 3. Install Active Directory Domain Services

Install Active Directory Domain Services and management tools:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

---

## 4. Create the Active Directory Forest

Create the `adlab.local` domain:

```powershell
Install-ADDSForest -DomainName "adlab.local" -DomainNetbiosName "ADLAB" -InstallDns
```

---

## 5. Verify the Domain

Check domain information:

```powershell
Get-ADDomain
```

Check forest information:

```powershell
Get-ADForest
```

Check the Domain Controller:

```powershell
Get-ADDomainController
```

Verify DNS resolution:

```powershell
Resolve-DnsName dc01.adlab.local
```

---

## 6. Import the Active Directory Module

Import the Active Directory PowerShell module:

```powershell
Import-Module ActiveDirectory
```

Store the domain Distinguished Name:

```powershell
$Base = (Get-ADDomain).DistinguishedName
```

---

## 7. Create Organizational Units

Create the users OU:

```powershell
New-ADOrganizationalUnit -Name "LabUsers" -Path $Base
```

Create the groups OU:

```powershell
New-ADOrganizationalUnit -Name "LabGroups" -Path $Base
```

Create the service accounts OU:

```powershell
New-ADOrganizationalUnit -Name "ServiceAccounts" -Path $Base
```

Create the workstations OU:

```powershell
New-ADOrganizationalUnit -Name "Workstations" -Path $Base
```

---

## 8. Create the Alice Domain User

Create Alice's password:

```powershell
$AlicePassword = ConvertTo-SecureString "ComplexP@ssw0rd2026!#Secure" -AsPlainText -Force
```

Create the domain user:

```powershell
New-ADUser -Name "Alice Carter" -GivenName "Alice" -Surname "Carter" -SamAccountName "alice" -UserPrincipalName "alice@adlab.local" -AccountPassword $AlicePassword -Enabled $true -PasswordNeverExpires $true -Path "OU=LabUsers,$Base"
```

---

## 9. Create the Helpdesk Group

```powershell
New-ADGroup -Name "Helpdesk" -SamAccountName "Helpdesk" -GroupCategory Security -GroupScope Global -Path "OU=LabGroups,$Base"
```

---

## 10. Create the SQL Service Account

Create the service-account password:

```powershell
$SqlPassword = ConvertTo-SecureString "Password123!" -AsPlainText -Force
```

Create the service account:

```powershell
New-ADUser -Name "SQL Service" -SamAccountName "svc_sql" -UserPrincipalName "svc_sql@adlab.local" -AccountPassword $SqlPassword -Enabled $true -PasswordNeverExpires $true -Path "OU=ServiceAccounts,$Base"
```

Add `svc_sql` to the Helpdesk group:

```powershell
Add-ADGroupMember -Identity "Helpdesk" -Members "svc_sql"
```

---

## 11. Configure the SQL Service Principal Name

Register the MSSQL SPN:

```powershell
setspn -S MSSQLSvc/sql01.adlab.local:1433 ADLAB\svc_sql
```

Enable RC4 for the controlled Kerberoasting lab:

```powershell
Set-ADUser -Identity "svc_sql" -Replace @{'msDS-SupportedEncryptionTypes'=4}
```

Verify the configuration:

```powershell
Get-ADUser svc_sql -Properties ServicePrincipalName,msDS-SupportedEncryptionTypes
```

---

## 12. Create the Backup Service Account

Create the password:

```powershell
$BackupPassword = ConvertTo-SecureString "BackupOnly2026!" -AsPlainText -Force
```

Create the account:

```powershell
New-ADUser -Name "Backup Service" -SamAccountName "backupsvc" -UserPrincipalName "backupsvc@adlab.local" -AccountPassword $BackupPassword -Enabled $true -PasswordNeverExpires $true -Path "OU=ServiceAccounts,$Base"
```

---

## 13. Create the Domain Administrator Account

Create the password:

```powershell
$DAPassword = ConvertTo-SecureString "DA-Only-2026!NeverReuse" -AsPlainText -Force
```

Create the account:

```powershell
New-ADUser -Name "Lab Domain Administrator" -SamAccountName "da_admin" -UserPrincipalName "da_admin@adlab.local" -AccountPassword $DAPassword -Enabled $true -Path "OU=LabUsers,$Base"
```

Add `da_admin` to Domain Admins:

```powershell
Add-ADGroupMember -Identity "Domain Admins" -Members "da_admin"
```

---

## 14. Configure Vulnerable Password Reset Delegation

Get the Distinguished Name of `backupsvc`:

```powershell
$BackupDN = (Get-ADUser backupsvc).DistinguishedName
```

Allow the Helpdesk group to reset the account password:

```powershell
dsacls $BackupDN /G "ADLAB\Helpdesk:CA;Reset Password"
```

Verify the delegated permission:

```powershell
dsacls $BackupDN | Select-String "Helpdesk"
```

---

## 15. Give backupsvc DCSync Permissions

Get the domain Distinguished Name:

```powershell
$DomainDN = (Get-ADDomain).DistinguishedName
```

Grant directory replication permission:

```powershell
dsacls $DomainDN /G "ADLAB\backupsvc:CA;Replicating Directory Changes"
```

Grant replication of all directory changes:

```powershell
dsacls $DomainDN /G "ADLAB\backupsvc:CA;Replicating Directory Changes All"
```

Grant filtered-set replication:

```powershell
dsacls $DomainDN /G "ADLAB\backupsvc:CA;Replicating Directory Changes In Filtered Set"
```

Verify the permissions:

```powershell
dsacls $DomainDN | Select-String "backupsvc"
```

---

## 16. Enable WinRM

```powershell
Enable-PSRemoting -Force
```

---

## 17. Allow svc_sql Remote Management Access

Get the Remote Management Users group:

```powershell
$RemoteManagementGroup = Get-ADGroup -Identity "S-1-5-32-580"
```

Add `svc_sql`:

```powershell
Add-ADGroupMember -Identity $RemoteManagementGroup -Members "svc_sql"
```

Verify membership:

```powershell
Get-ADGroupMember -Identity "S-1-5-32-580"
```

---

## 18. Enable Security Auditing

Enable Kerberos service-ticket auditing:

```powershell
auditpol /set /subcategory:"Kerberos Service Ticket Operations" /success:enable /failure:enable
```

Enable user-account management auditing:

```powershell
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
```

Enable Active Directory access auditing:

```powershell
auditpol /set /subcategory:"Directory Service Access" /success:enable /failure:enable
```

Enable logon auditing:

```powershell
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

---

## 19. Review Security Events

Review important events:

```powershell
Get-WinEvent -FilterHashtable @{LogName="Security";Id=4624,4724,4769,4662} -MaxEvents 100 | Select-Object TimeCreated,Id,Message
```

Review Kerberos events associated with `svc_sql`:

```powershell
Get-WinEvent -FilterHashtable @{LogName="Security";Id=4769} | Where-Object {$_.Message -match "svc_sql"} | Select-Object TimeCreated,Id,Message
```
