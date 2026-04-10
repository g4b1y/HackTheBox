# Explosion - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Windows
**IP Address:** 10.129.1.13
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Explosion**. 

*   **Initial Access:** Unauthenticated RDP access as `Administrator` (Port 3389).
*   **Privilege Escalation:** Not required as initial access provided administrative privileges immediately.
*   **Key Learning Points:** 
    * Identifying and exploitation of misconfigured RDP services.
    * Using `xfreerdp` for remote desktop connections.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 135/tcp | msrpc | Microsoft Windows RPC | Standard Windows RPC service. |
| 139/tcp | netbios-ssn | Microsoft Windows netbios-ssn | NetBIOS session service. |
| 445/tcp | microsoft-ds | - | SMB service. |
| 3389/tcp | ms-wbt-server | Microsoft Terminal Services | RDP service, potentially misconfigured. |
| 5985/tcp | http | Microsoft HTTPAPI httpd 2.0 | WinRM service. |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/Explosion 10.129.1.13
```

## 2.2. Web Enumeration
**Strategy:** No public web services were identified on standard ports.

---

# 3. Initial Access (Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Unauthenticated RDP login for the `Administrator` account.
*   **Vector:** RDP service on port 3389.

## 3.2. Exploitation Path
The reconnaissance phase revealed an open RDP service on port 3389. Testing for common misconfigurations, an attempt was made to connect as the `Administrator` user without a password. The connection was successful, granting direct access to the Windows desktop environment. The `flag.txt` file was located on the Administrator's desktop.

**Payload/Tool:**
```bash
# Connect via RDP
xfreerdp /u:Administrator /cert:ignore /v:10.129.1.13
# Login with blank password
```

**Flag:**
```bash
# 951fa96d7830c451b536be5a6be008a0
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Not required as the initial session was already `Administrator`.

---

# 5. Credentials & Loot

| Username | Password | Service | Note |
| :--- | :--- | :--- | :--- |
| Administrator | (blank) | RDP | Direct administrative access. |

---

# 6. Recommendations & Mitigation
1. **Enforce Strong Passwords:** Set a complex password for the `Administrator` account and all other user accounts.
2. **Disable Unnecessary Services:** If RDP is not required for remote administration, it should be disabled.
3. **Use Network Level Authentication (NLA):** Enable NLA to require authentication before an RDP session is established.
4. **Restrict Access:** Use a firewall or Remote Desktop Gateway to limit RDP access to specific authorized IP addresses.
