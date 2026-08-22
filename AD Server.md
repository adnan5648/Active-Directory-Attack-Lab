**Organizational Units**

At the beginning of the lab, the Active Directory structure was organized by creating dedicated Organizational Units for LabUsers, LabGroups, ServiceAccounts, and Workstations. These OUs provide a clean structure for placing the users, groups, service accounts, and domain-joined systems that will later be used in the vulnerable AD environment.

<img width="861" height="172" alt="image" src="https://github.com/user-attachments/assets/8692d47a-4bbd-43b8-a79b-1e89498db3c6" />

After creation, the Organizational Units were verified using PowerShell to confirm that they existed under the adlab.local domain and that their Distinguished Names were configured correctly.

<img width="896" height="560" alt="image" src="https://github.com/user-attachments/assets/bf4dcc56-55b7-45f6-95f4-344ce502e93f" />

**Low Privilege User**

A standard domain user, Alice Carter (alice@adlab.local), was created inside the LabUsers Organizational Unit. 

A lab-only password was converted into a secure string and assigned to the account, which was then enabled for use in the controlled Active Directory environment.

<img width="1022" height="62" alt="image" src="https://github.com/user-attachments/assets/7d87bc70-40d6-4d21-bec2-139423493070" />

<img width="1097" height="92" alt="image" src="https://github.com/user-attachments/assets/3f48aa91-c8f9-4ca5-a444-5493bcfd7339" />

**Helpdesk Group**

A security group named Helpdesk was created inside the LabGroups Organizational Unit. This group will later be used to assign delegated permissions to selected accounts, forming part of the vulnerable privilege structure in the lab.

<img width="1095" height="97" alt="image" src="https://github.com/user-attachments/assets/d7d84367-5489-46f0-acc8-9eb48cc1a135" />

**Vulnerable Service Account**

A deliberately weak service account named svc_sql was created inside the ServiceAccounts Organizational Unit. The account uses a predictable lab-only password and is configured not to expire, making it suitable for demonstrating service-account abuse and Kerberoasting in the controlled environment.

<img width="1097" height="107" alt="image" src="https://github.com/user-attachments/assets/b132b653-378c-4f64-8d47-49ac2706898e" />

**Vulnerable Account to HelpDesk Group**

The svc_sql service account was added to the Helpdesk security group. This gives the service account additional delegated privileges, creating an excessive group membership weakness that can later be abused during privilege escalation.

<img width="837" height="76" alt="image" src="https://github.com/user-attachments/assets/15a4a43e-6e0f-4d22-96f7-fe2665295b81" />

**SPN against the account**

A Service Principal Name (SPN) for MSSQLSvc/sql01.adlab.local:1433 was registered to the svc_sql service account. 
This makes the account Kerberoastable, allowing authenticated domain users to request a Kerberos service ticket that can later be tested offline against the account’s weak password.

<img width="926" height="173" alt="image" src="https://github.com/user-attachments/assets/bbc8f343-b11d-498b-87d9-afaf968f17c6" />

Now MSSQL Service owned by svc_sql user.

**Configure RC4 Encryption**

<img width="1026" height="82" alt="image" src="https://github.com/user-attachments/assets/10498e27-8f88-4a2c-bb35-fefbbbd7b690" />

The svc_sql account was configured to support RC4 encryption by setting msDS-SupportedEncryptionTypes to 4. This weakens the Kerberos configuration and makes the service ticket easier to use in a controlled Kerberoasting demonstration within the lab.

**Verifying RC4 Configuration**

<img width="1092" height="230" alt="image" src="https://github.com/user-attachments/assets/4b829159-5d63-40a5-a5d8-6a24e9b827e6" />

The svc_sql account configuration was verified by checking its registered Service Principal Name (SPN) and msDS-SupportedEncryptionTypes attribute.

**Vulnerable Backup Account**

<img width="1340" height="126" alt="image" src="https://github.com/user-attachments/assets/b8f6c727-b9c6-4c14-8caf-4d5d7e8d783a" />

A service account named backupsvc was created inside the ServiceAccounts Organizational Unit using a lab-only password. This account will later be assigned excessive Active Directory replication permissions, making it part of the privilege-escalation and DCSync attack path

**Dedicated Domain Administrator**

<img width="1117" height="232" alt="image" src="https://github.com/user-attachments/assets/edf73388-8789-4e90-9c45-3c6621e4b830" />

Dedicated domain administrator account, da_admin, was created inside the LabUsers Organizational Unit using a lab-only password.

**Add Account to Domain Admins Group**

<img width="1002" height="72" alt="image" src="https://github.com/user-attachments/assets/41988c71-e695-4761-85f8-fcef97ec14b6" />

The da_admin account was added to the built-in Domain Admins group, granting it full administrative privileges across the Active Directory domain.

**Password Reset Permission to Helpdesk**

<img width="850" height="422" alt="image" src="https://github.com/user-attachments/assets/f317b863-f774-44f0-ac31-72e2858dd0f5" />

The command retrieves the Distinguished Name of the backupsvc account and uses dsacls to grant the ADLAB\Helpdesk group permission to reset its password.

**Active Directory Replication Permission**

<img width="632" height="440" alt="image" src="https://github.com/user-attachments/assets/2f5e094e-749c-4cf9-bf08-d43e855ee4ea" />

The command retrieves the domain’s Distinguished Name and uses dsacls to grant the backupsvc account the Replicating Directory Changes permission on the adlab.local domain.

<img width="717" height="377" alt="image" src="https://github.com/user-attachments/assets/7bdf1021-b5ba-4e62-9f7f-a1b0c0ef937d" />

The command uses dsacls to grant the backupsvc account the Replicating Directory Changes All permission on the adlab.local domain, allowing replication of all directory changes, including sensitive data.

<img width="756" height="382" alt="image" src="https://github.com/user-attachments/assets/d1971ca7-2a7c-40ad-adb3-23e3b7062065" />

The command uses dsacls to grant the backupsvc account the Replicating Directory Changes In Filtered Set permission on the adlab.local domain, allowing replication of attributes included in the filtered attribute set.

**WinRM Configuration**

<img width="495" height="62" alt="image" src="https://github.com/user-attachments/assets/176f738c-6d3e-45ac-83ae-b2e95a0ce34b" />

The hostname command confirms the system is DC01, and Enable-PSRemoting -Force enables PowerShell Remoting on the domain controller so it can accept remote PowerShell connections.

**svc_sql user to Remote Management Group**

<img width="877" height="87" alt="image" src="https://github.com/user-attachments/assets/258ac9b3-83ac-4234-bbfb-023d7a04fd08" />

Command adds the svc_sql service account to that group, allowing it to use Windows remote management services where permitted.

**Verify Remote Management Users Membership**

<img width="727" height="225" alt="image" src="https://github.com/user-attachments/assets/f403d753-373a-4d0e-91e0-1eeb72312286" />


The command lists the members of the built-in Remote Management Users group. The output confirms that the svc_sql service account was successfully added to the group.
