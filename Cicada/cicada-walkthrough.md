# Cicada - HackTheBox Writeup

**Date:** 2026-03-19
**OS:** Windows Server 2022
**IP Address:** 10.129.231.149
**Difficulty:** Easy
**Points:** 20

---

# 1. Executive Summary

This writeup documents the comprehensive exploitation process for the HackTheBox machine **Cicada**. The attack path demonstrates the critical impact of simple information disclosure and the accumulation of low-level misconfigurations in an Active Directory environment.

*   **Initial Access:** Achieved by discovering a default password in an anonymously accessible SMB share (`HR`). This led to a successful password spray and subsequent exploitation of sensitive information leaked in LDAP user descriptions and internal backup scripts.
*   **Privilege Escalation:** Facilitated by the `SeBackupPrivilege` assigned to the `Backup Operators` group. This allowed for bypassing standard NTFS file system security to read the root flag directly from the Administrator's desktop.
*   **Key Learning Points:** 
    *   **Public Information Disclosure:** Default passwords in internal notices create an immediate entry point.
    *   **AD Metadata Security:** User descriptions should never be used to store sensitive information like passwords.
    *   **High-Impact Privileges:** Membership in administrative groups like `Backup Operators` provides full system compromise capability through built-in Windows features.

---

# 2. Reconnaissance & Enumeration Strategy

## 2.1. Methodology Overview
The enumeration strategy followed a "Defense-in-Depth in Reverse" approach, peeling back layers of the Active Directory environment:
1.  **Passive & Broad Scan**: Identifying all open services and their versions.
2.  **Unauthenticated Probing**: Looking for null sessions and guest access (SMB/RPC).
3.  **Credential Pivoting**: Using findings from one layer to access the next (e.g., Guest -> Michael -> David -> Emily).
4.  **Strategic Privilege Abuse**: Leveraging system-level groups and privileges (Backup Operators) for final compromise.

## 2.2. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 53 | DNS | Simple DNS Plus | Domain Name Service |
| 88 | Kerberos | Microsoft Windows Kerberos | Authentication Service |
| 135 | RPC | Microsoft Windows RPC | Remote Procedure Call |
| 389 | LDAP | Microsoft Windows Active Directory LDAP | Domain Controller |
| 445 | SMB | Microsoft Windows Server 2008 R2 - 2012 microsoft-ds | Active Directory SMB |
| 5985 | WinRM | Microsoft Windows Remote Management | Remote Shell access |

**Nmap Command:**
```bash
nmap -p- -sV -sC 10.129.231.149 -oN nmap/cicada-full.nmap
```

## 2.3. SMB Enumeration
Using `netexec` (formerly CrackMapExec), we identified shares accessible without authentication.
```bash
nxc smb 10.129.231.149 -u 'guest' -p '' --shares
```
**The Finding Strategy**: I focused on unauthenticated SMB first to find "low-hanging fruit" like onboarding documents or notices. The `HR` share was accessible with `READ` permissions for the `Guest` account. Inside, `Notice from HR.txt` contained a cleartext password.
**Credential Found**: `Cicada$M6Corpb*@Lp#nZp!8`

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Information Disclosure and Weak Password Management.
*   **Vector:** Anonymously accessible file share and hardcoded passwords in scripts and AD attributes.

## 3.2. Exploitation Path
1.  **User Enumeration**: Performed a RID brute-force via the null session to identify valid domain users.
    **The Logic**: Since null sessions were enabled, I used `netexec` to cycle through Relative Identifiers (RIDs), forcing the server to reveal names associated with internal SIDs.
    ```bash
    nxc smb 10.129.231.149 -u 'guest' -p '' --rid-brute
    ```
2.  **Password Spraying**: Sprayed the HR password against the target list.
    **The Logic**: A single valid password found in an HR notice is often a "default" for many users. Testing it once against all accounts (spraying) avoids account lockouts.
    ```bash
    nxc smb 10.129.231.149 -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8'
    ```
3.  **LDAP Enumeration**: Used Michael's credentials to dump domain user descriptions.
    **The Logic**: Domain user descriptions are often used by admins for "temporary" notes. I specifically targeted these metadata fields for credential leaks.
    ```bash
    nxc ldap 10.129.231.149 -u 'michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
    ```
    **Leak Found**: David Orelious had his password in his description field.
4.  **Lateral Movement**: Logged into the `DEV` share as David and retrieved `Backup_script.ps1`.
    **The Logic**: New credentials open new shares. I re-enumerated all SMB shares as David to find the `DEV` directory.
5.  **Emily Oscars' Credentials**: Extracted Emily Oscars' WinRM password from the hardcoded script: `Q!3@Lp#M6b*7t*Vt`.
6.  **WinRM Shell**: Established an initial foothold for the user flag using `evil-winrm`.

**User Flag:**
```bash
type C:\Users\emily.oscars.CICADA\Desktop\user.txt
# f1a16cbf14c508847221225dbd22edbf
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
A comprehensive privilege check revealed Emily as a member of the **`Backup Operators`** group with **`SeBackupPrivilege`** enabled.
```powershell
whoami /all
```

## 4.2. Exploitation Path
**The Strategic Decision**: Membership in `Backup Operators` is effectively full-system access. It allows bypassing NTFS ACLs by using the "Backup" mode. I used `robocopy /B` to copy the root flag, effectively telling the kernel to ignore security checks for this "backup task."
```powershell
robocopy /B C:\Users\Administrator\Desktop C:\Users\emily.oscars.CICADA root.txt
```

**Vulnerability:** Misconfigured High-Privilege Group Membership.
**Payload/Tool:** `robocopy /B` (Built-in Windows tool).

**Root Flag:**
```bash
type C:\Users\emily.oscars.CICADA\root.txt
# 438b711e7c8477a357fbe7f307f1da14
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source | Notes |
| :--- | :--- | :--- | :--- |
| michael.wrightson | Cicada$M6Corpb*@Lp#nZp!8 | HR Notice | Default/Corporate Password |
| david.orelious | aRt$Lp#7t*VQ!3 | LDAP Description | Sensitive note in AD |
| emily.oscars | Q!3@Lp#M6b*7t*Vt | Backup_script.ps1 | Hardcoded in Dev Script |
| Administrator | 2b87e6c991581e1c9a63513c2022020a | SAM Hive Dump | Local Admin NT Hash |
| **User Flag** | **f1a16cbf14c508847221225dbd22edbf** | Emily's Desktop | Obtained via WinRM |
| **Root Flag** | **438b711e7c8477a357fbe7f307f1da14** | Admin's Desktop | Obtained via SeBackupPrivilege |

---

# 6. Recommendations & Mitigation
1. **Remediate Information Leaks**: 
    - **Audit Shares**: Ensure internal notices and default passwords are not stored in accessible locations.
    - **Clean AD Metadata**: Prohibit storing passwords in user description fields.
2. **Secure Scripting Practices**:
    - **Credential Safety**: Use environment variables, secure strings, or secret management solutions instead of hardcoding passwords in backup or dev scripts.
3. **Principle of Least Privilege**:
    - **Review High-Impact Groups**: Membership in groups like `Backup Operators` should be strictly controlled and audited regularly.
    - **Restrict WinRM**: Limit WinRM access to required users and administrative hosts only.
4. **Monitoring & Alerting**:
    - **Privilege Usage**: Alert on the activation and use of `SeBackupPrivilege` by non-system accounts.
    - **Enumeration Patterns**: Monitor for unusual LDAP queries or RID brute-force attempts from unauthorized IPs.
