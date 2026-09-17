# Active Directory Vulnerabilities

## 1. Weak Kerberoastable Service Account

**Definition:** A service account with a registered SPN and a weak password can be targeted through Kerberoasting, allowing an authenticated domain user to request and crack its Kerberos service ticket offline.

**Impact:** Compromise of the service account may expose additional systems, services, and privileges.

**MITRE ATT&CK:** `T1558.003 - Kerberoasting`

---

## 2. Excessive Group Membership

**Definition:** An account is assigned to a security group that provides more permissions than required. In this lab, `svc_sql` is added to the `Helpdesk` group.

**Impact:** If the account is compromised, the attacker also gains the additional permissions inherited from the group.

**MITRE ATT&CK:** `Privilege Escalation / Abuse of Permissions`

---

## 3. Delegated Password Reset Permission

**Definition:** The `Helpdesk` group is allowed to reset the password of the `backupsvc` account, creating an unsafe privilege relationship.

**Impact:** An attacker controlling a Helpdesk member may take over `backupsvc` without knowing its original password.

**MITRE ATT&CK:** `T1098 - Account Manipulation`

---

## 4. Excessive Active Directory Replication Rights

**Definition:** The `backupsvc` account is intentionally granted directory replication permissions normally reserved for highly trusted systems.

**Impact:** These permissions can be abused through DCSync to retrieve sensitive credential data and potentially compromise the domain.

**MITRE ATT&CK:** `T1003.006 - DCSync`

---

## 5. Remote Management Access for Service Account

**Definition:** The `svc_sql` account is permitted to access the Domain Controller through Windows Remote Management.

**Impact:** If compromised, the service account may be used for remote access and lateral movement between systems.

**MITRE ATT&CK:** `T1021.006 - Windows Remote Management`

---

## 6. Weak Password Policy

**Definition:** Lab accounts intentionally use weak or predictable passwords that are easier to guess or crack.

**Impact:** Weak passwords increase the risk of credential compromise and unauthorized access to domain resources.

**MITRE ATT&CK:** `Credential Access`
