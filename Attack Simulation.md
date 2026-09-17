# Initial Access
The assessment began with credentials for a standard domain user, simulating a successful credential theft or phishing event. Phishing was not performed.

**Connect to WS01 as Alice**

<img width="1258" height="267" alt="image" src="https://github.com/user-attachments/assets/8763f718-4a28-4b18-a3f7-f7014032807c" />

Connects to the Windows 11 workstation WS01 using Alice’s valid domain credentials through WinRM.

<img width="794" height="177" alt="image" src="https://github.com/user-attachments/assets/e4d264d6-b717-46f5-95fe-6c4b677e89fa" />

Verifies the current user context, hostname, group memberships, and network configuration to confirm successful initial access.

<img width="759" height="238" alt="image" src="https://github.com/user-attachments/assets/160f2f89-f1bc-4221-bb67-681846f828d1" />

<img width="1028" height="539" alt="image" src="https://github.com/user-attachments/assets/12988535-3c53-4783-a97e-26874405253f" />

# Domain Enumeration

<img width="1267" height="276" alt="image" src="https://github.com/user-attachments/assets/2dfa9970-5fd4-43e9-8612-5ff3199e0990" />

Confirms that DC01.adlab.local (10.0.0.10) is the active domain controller for the adlab.local domain.

<img width="964" height="389" alt="image" src="https://github.com/user-attachments/assets/d16fc1ef-0c84-4131-8749-9280d0dfceee" />

<img width="385" height="277" alt="image" src="https://github.com/user-attachments/assets/b8b8cd79-46bf-4a1e-abbd-62b0f7452721" />

Nmap Scans both DC01 (10.0.0.10) and WS01 (10.0.0.20) for common Active Directory and remote-management ports.
The scan helps identify reachable services such as Kerberos, LDAP, SMB, DNS, and WinRM that may be relevant to later attack stages.

**Credential Access through Kerberoasting**

<img width="1253" height="260" alt="image" src="https://github.com/user-attachments/assets/2d6b6c42-be73-42bf-80c3-33398c8962cf" />

Queries the domain for accounts with registered Service Principal Names (SPNs) and identifies svc_sql as a Kerberoastable service account.
The account is also a member of the Helpdesk group, making it relevant for the next privilege-escalation stage.

<img width="486" height="89" alt="image" src="https://github.com/user-attachments/assets/f46d177b-1caf-45cb-b293-d0ee9cdb8c24" />

Opeing the kerbroasting file gives the following output.

<img width="865" height="137" alt="image" src="https://github.com/user-attachments/assets/420d607a-7ef3-46a0-a956-cfd024e7ddfb" />

**Cracking Kerberoas Ticket**
 
 <img width="1162" height="30" alt="image" src="https://github.com/user-attachments/assets/7d6abdc3-0102-4461-83a3-7deb1b1d2f1f" />

 Creates a small custom wordlist containing likely service-account passwords for the lab environment.

<img width="316" height="185" alt="image" src="https://github.com/user-attachments/assets/dced5086-86ef-4691-92b6-3b0c38d47381" />

This wordlist is used in the next step to perform an offline password-cracking attempt against the captured Kerberos service ticket.

**Cracking the Kerberoasted TGS Hash**

Uses Hashcat to perform an offline cracking attack against the captured Kerberos TGS ticket for svc_sql.

<img width="810" height="151" alt="image" src="https://github.com/user-attachments/assets/d9b2004f-ac39-46b0-a6b9-0c6dbec2adaf" />

<img width="995" height="267" alt="image" src="https://github.com/user-attachments/assets/a459c047-dade-4050-b146-123ef8f0c1d1" />

The attack successfully recovers the service account password as Password123!, confirming the Kerberoasting attack is successful.

# Privilege Escalation

**Using Delegated Password Reset**

Uses the compromised svc_sql account to reset the password of the more privileged backupsvc account.

<img width="1243" height="211" alt="image" src="https://github.com/user-attachments/assets/73bdbf2c-dfae-456e-a483-d3707cd26b94" />

The new backupsvc credentials are then validated successfully against the domain, confirming control of the account.

<img width="1251" height="220" alt="image" src="https://github.com/user-attachments/assets/3de2e5df-0c4f-4e36-884a-0865816b52d2" />

# Lateral Movement via WinRM

<img width="876" height="323" alt="image" src="https://github.com/user-attachments/assets/beb9a7b6-ecf5-4456-9cad-640b701f7d72" />

Uses the compromised svc_sql credentials to establish a remote WinRM session on DC01.

<img width="651" height="96" alt="image" src="https://github.com/user-attachments/assets/bb743f82-4fa5-4d31-9f89-c778f81ab833" />

The whoami and hostname commands confirm successful access as ADLAB\svc_sql on the domain controller.

# Domain Compromise via DCSync

<img width="1240" height="365" alt="image" src="https://github.com/user-attachments/assets/e5419811-689c-45f5-9d0e-7f6ddbed1ed1" />

Uses the compromised backupsvc account to perform a DCSync attack and retrieve the credentials of the privileged da_admin account.
This confirms that the delegated replication rights can expose domain administrator credential material and lead to full domain compromise.

**After obtaining the da_admin NTLM hash, the attacker can potentially authenticate as the Domain Administrator without knowing the plaintext password.
This gives access to highly privileged domain resources and administrative functions.
From this point, the attacker could modify accounts, access sensitive systems, and establish persistence across the domain.**
