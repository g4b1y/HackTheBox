# [Machine Name] - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Linux
**IP Address:** 10.129.15.45
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Appointment**. 

*   **Initial Access:** Gained by exploiting a SQL Injection vulnerability in the login form using a tautology payload.
*   **Privilege Escalation:** Not required for this "Starting Point" machine as the initial access provided the flag directly.
*   **Key Learning Points:** 
    * Understanding SQL Injection (SQLi) authentication bypass.
    * Identifying web server versions and potential attack vectors.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 80/tcp | HTTP | Apache httpd 2.4.38 ((Debian)) | Web Login Page |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/initial 10.129.15.45
```

## 2.2. Web Enumeration
**Strategy:** Direct inspection of the login page and testing for common authentication bypass vulnerabilities.

No additional subdomains or virtual hosts were relevant for this lab.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** SQL Injection (Auth Bypass)
*   **Vector:** Login Form (Username/Password fields)

## 3.2. Exploitation Path
The login form was found to be vulnerable to SQL injection. By entering a tautology in the username or password field, the back-end SQL query evaluates to true, bypassing the necessity of a valid password.

**Payload/Tool:**
```sql
admin' OR 1=1#
```

**User Flag:**
```text
e3d0796d002a446c0e6222226f42e9672
```

---

# 4. Privilege Escalation (Root Flag)

This machine is a single-level challenge; no further privilege escalation was required once the admin dashboard was accessed.

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| admin | ! OR 1 = ! | SQL Injection |

---

# 6. Recommendations & Mitigation
1. **Parameterized Queries:** Use prepared statements and parameterized queries to prevent user input from being executed as SQL code.
2. **Input Validation:** Implement strict input validation to ensure that only expected characters and formats are accepted.
3. **Least Privilege:** Ensure the database user has the minimum necessary permissions.


