# Preignition - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Linux
**IP Address:** 10.129.14.162
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Preignition**. 

*   **Initial Access:** Discovery of a hidden administrative login page through directory brute-forcing.
*   **Privilege Escalation:** Not required for this Tier 0 machine.
*   **Key Learning Points:** 
    * Web directory enumeration using `gobuster`.
    * Identifying and accessing administrative panels with default credentials.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 80/tcp | http | nginx | Web server is accessible. |

**Nmap Command:**
```bash
nmap -p- --min-rate=5000 10.129.14.162 -oN nmap/Preignition-allports
```

![Default Nginx Page](file:///home/rf2i/Documents/HackTheBox/Starting-Point/Preignition/images/image.png)

## 2.2. Web Enumeration
**Strategy:** Directory brute-forcing to identify hidden files and directories.

### Directory Brute Forcing
The initial scan showed only a default Nginx landing page. A directory search was performed using `gobuster` with a common wordlist and checking for `.php` extensions.

```bash
gobuster dir -u http://10.129.14.162 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php
```

**Results:**
- `admin.php` (Status: 200)

---

# 3. Initial Access (Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Use of default/weak credentials on an administrative panel.
*   **Vector:** `admin.php` login form.

## 3.2. Exploitation Path
Navigating to `http://10.129.14.162/admin.php` revealed a login console. Testing for default credentials, the combination `admin:admin` was successful.

![Admin Login Panel](file:///home/rf2i/Documents/HackTheBox/Starting-Point/Preignition/images/image-1.png)

Upon successful login, the administrative dashboard displayed the flag.

![Admin Dashboard with Flag](file:///home/rf2i/Documents/HackTheBox/Starting-Point/Preignition/images/image-2.png)

**Flag:**
```bash
# 6483bee07c1c1d57f14e5b0717503c73
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Not applicable for this target.

---

# 5. Credentials & Loot

| Username | Password | Source | Note |
| :--- | :--- | :--- | :--- |
| admin | admin | Manual | Default credentials for the admin panel. |

---

# 6. Recommendations & Mitigation
1. **Change Default Credentials:** Immediately change the default `admin` password to a strong, unique one.
2. **Restrict Access to Admin Panels:** Move the administrative panel to a non-standard path or restrict access via IP whitelisting or additional authentication (e.g., HTBasic Auth).
3. **Disable Directory Listing/Information Leakage:** Ensure that service versions and default pages are not exposing unnecessary information about the server.