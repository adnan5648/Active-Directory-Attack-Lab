# Windows 11 - WS01

> This workstation acts as the initial compromised endpoint in the isolated Active Directory lab.

---

## 1. Check the Network Adapter

```powershell
Get-NetAdapter
```

---

## 2. Configure Static IP

Configure the workstation IP address:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "10.0.0.20" -PrefixLength 24
```

Configure DC01 as the DNS server:

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "10.0.0.10"
```

---

## 3. Test Domain Controller Connectivity

Test connectivity:

```powershell
ping 10.0.0.10
```

Verify DNS resolution:

```powershell
Resolve-DnsName dc01.adlab.local
```

---

## 4. Join WS01 to the Domain

Enter Domain Administrator credentials:

```powershell
$Credential = Get-Credential "ADLAB\Administrator"
```

Join the workstation to `adlab.local`:

```powershell
Add-Computer -DomainName "adlab.local" -Credential $Credential -OUPath "OU=Workstations,DC=adlab,DC=local" -NewName "WS01" -Restart
```

---

## 5. Enable WinRM

```powershell
Enable-PSRemoting -Force
```

---

## 6. Allow Alice to Use WinRM

Add Alice to the local Remote Management Users group:

```powershell
Add-LocalGroupMember -Group "Remote Management Users" -Member "ADLAB\alice"
```

Verify membership:

```powershell
Get-LocalGroupMember "Remote Management Users"
```

---

## 7. Verify User and System Information

Check the current user:

```powershell
whoami
```

Check the hostname:

```powershell
hostname
```

Check group memberships:

```powershell
whoami /groups
```

Display detailed user information:

```powershell
whoami /all
```

Display network configuration:

```powershell
ipconfig /all
```

---

## 8. Identify the Domain Controller

```powershell
nltest /dsgetdc:adlab.local
```

---

## 9. Enumerate Domain Accounts

Enumerate domain users:

```powershell
net user /domain
```

Enumerate domain groups:

```powershell
net group /domain
```

Check Domain Administrators:

```powershell
net group "Domain Admins" /domain
```

Discover domain systems:

```powershell
net view /domain
```
