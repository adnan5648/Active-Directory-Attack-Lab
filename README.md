# Vulnerable Active Directory Lab

**Overview**

This project is a deliberately vulnerable Microsoft Active Directory lab built for penetration-testing and red-team practice.

The environment contains a Windows Domain Controller, a domain-joined Windows workstation, and a Kali Linux attacker machine. Several Active Directory misconfigurations were intentionally introduced to study common privilege-escalation and credential-access techniques in a controlled environment.

**Note:** This lab is isolated from the main network and is intended only for authorized cybersecurity training.

## Project Documentation

- [AD Server Setup](01-AD-Server-Setup.md)
- [Windows 11 Setup](02-Windows-11-Setup.md)
- [Attack Simulation](03-Attack-Simulation.md)
- [Intentional Misconfigurations](04-Intentional-Vulnerabilities.md)
- [MITRE ATT&CK Mapping](MITRE/Mapping.md)
- [Command Reference](Commands/)
- [Lab Architecture](Lab%20Architecture/)

  
## Active Directory Structure

The domain contains dedicated Organizational Units for:

- `LabUsers`
- `LabGroups`
- `ServiceAccounts`
- `Workstations`

### Lab Accounts

- `alice`
- `svc_sql`
- `backupsvc`
- `da_admin`

# Intentional Vulnerabilities
  The environment has following vulnerabilities

- `Weak Kerberoastable service account`
- `Excessive group membership`
- `Delegated password reset permissions`
- `Excessive Active Directory replication rights`
- `Unnecessary remote management access`
- `Weak passwords`

Detailed explanations are available in [View intentional vulnerabilities](04-Intentional-Vulnerabilities.md).

# Attack Scenario

<img width="1122" height="1402" alt="image" src="https://github.com/user-attachments/assets/6c83a82d-fd84-4ac5-b891-63ab65e3c68f" />

# Disclaimer

This project is intended strictly for educational purposes and authorized security testing.

All systems, accounts, passwords, permissions, and vulnerabilities used in this project are synthetic and exist only inside an isolated lab environment.
