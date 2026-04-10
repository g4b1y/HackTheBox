# Fawn - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Linux
**IP Address:** 10.129.14.158
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Fawn**. 

*   **Initial Access:** Anonymous FTP login (Port 21).
*   **Privilege Escalation:** Not required for this Tier 0 machine as the goal was to access the FTP server and retrieve the flag.
*   **Key Learning Points:** 
    * Identifying and exploitation of insecure FTP configurations (anonymous access).
    * Familiarization with FTP commands (`ls`, `get`).

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 21/tcp | ftp | vsftpd 3.0.3 | Anonymous login allowed. |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/Fawn 10.129.14.158
```

## 2.2. Web Enumeration
**Strategy:** No web services were found on the target machine.

---

# 3. Initial Access (Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Anonymous FTP login allowed on `vsftpd 3.0.3`.
*   **Vector:** FTP service on port 21.

## 3.2. Exploitation Path
The reconnaissance phase identified an FTP service running on port 21 with anonymous login enabled. By connecting to the service and providing `anonymous` as the username and an empty password (or any string), access to the FTP file system was granted. The `flag.txt` file was found in the root directory and downloaded.

**Payload/Tool:**
```bash
ftp 10.129.14.158
# Name: anonymous
# Password: [ENTER]
# ftp> ls
# ftp> get flag.txt
```

**Flag:**
```bash
cat flag.txt
# 035db21c881520061c53e0536e44f815
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Not applicable for this target as the primary goal was met via FTP access.

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| anonymous | (blank) | FTP Anonymous Login |

---

# 6. Recommendations & Mitigation
1. **Disable Anonymous FTP:** Modify the FTP configuration (e.g., `/etc/vsftpd.conf`) to set `anonymous_enable=NO`.
2. **Use Secure Protocols:** Replace FTP with SFTP (SSH File Transfer Protocol) or FTPS to ensure all data and credentials are encrypted during transit.
3. **Account Security:** If FTP must be used, ensure all users are authenticated with strong passwords.
