# Vulnerable Active Directory Lab

**Overview**

This project is a deliberately vulnerable Microsoft Active Directory lab built for penetration-testing and red-team practice.

The environment contains a Windows Domain Controller, a domain-joined Windows workstation, and a Kali Linux attacker machine. Several Active Directory misconfigurations were intentionally introduced to study common privilege-escalation and credential-access techniques in a controlled environment.

**Note:** This lab is isolated from the main network and is intended only for authorized cybersecurity training.

## Lab Architecture

| Machine | Role | IP Address |
|---|---|---|
| `DC01` | Domain Controller | `10.0.0.10` |
| `WS01` | Domain Workstation | `10.0.0.20` |
| `KALI` | Penetration Testing Machine | `10.0.0.50` |

**Domain:** `adlab.local`

All systems communicate through an isolated virtual LAN segment, keeping the intentionally vulnerable environment separated from the main network.

---

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

Detailed explanations are available in [`Vulnerabilities.md`](Vulnerabilities.md).

# Attack Scenario

<img width="1122" height="1402" alt="image" src="https://github.com/user-attachments/assets/6c83a82d-fd84-4ac5-b891-63ab65e3c68f" />

# Disclaimer

This project is intended strictly for educational purposes and authorized security testing.

All systems, accounts, passwords, permissions, and vulnerabilities used in this project are synthetic and exist only inside an isolated lab environment.
