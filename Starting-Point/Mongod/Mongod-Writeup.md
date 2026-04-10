# Mongod - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Linux
**IP Address:** 10.129.14.164
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Mongod**. 

*   **Initial Access:** Unauthenticated MongoDB database access (Port 27017).
*   **Privilege Escalation:** Not required for this Tier 0 machine.
*   **Key Learning Points:** 
    * Enumerating MongoDB databases using `mongosh`.
    * Identifying and retrieving sensitive data from misconfigured NoSQL databases.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 22/tcp | ssh | OpenSSH 7.6p1 | Standard SSH service. |
| 27017/tcp | mongodb | MongoDB 3.6.8 | Unauthenticated access allowed. |

**Nmap Command:**
```bash
nmap -p 22,27017 -sC -sV -oN nmap/Mongod 10.129.14.164
```

## 2.2. Web Enumeration
**Strategy:** No web services were found on the target machine.

---

# 3. Initial Access (Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Unauthenticated MongoDB database access.
*   **Vector:** MongoDB service on port 27017.

## 3.2. Exploitation Path
The reconnaissance phase identified a MongoDB service on port 27017. Using the `mongosh` tool, a connection was established to the remote host. The server's startup warnings confirmed that access control was not enabled, allowing unrestricted read and write access. By listing the available databases, a database named `sensitive_information` was discovered. Inside this database, a collection named `flag` contained the objective.

**Payload/Tool:**
```bash
# Connect to MongoDB
./mongosh mongodb://10.129.14.164:27017

# Enumerate and retrieve flag
test> show dbs
test> use sensitive_information
sensitive_information> show collections
sensitive_information> db.flag.find()
```

**Flag:**
```bash
# 1b6e6fb359e7c40241b6d431427ba6ea
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Not applicable for this target.

---

# 5. Credentials & Loot

| Database | Collection | Note |
| :--- | :--- | :--- |
| sensitive_information | flag | Contains the machine flag. |

---

# 6. Recommendations & Mitigation
1. **Enable Authentication:** Configure MongoDB to use Role-Based Access Control (RBAC) by enabling security in the `mongod.conf` file.
2. **Bind to localhost:** Restrict MongoDB to listen only on the loopback interface (`127.0.0.1`) if remote access is not required.
3. **Use a Firewall:** Use a network firewall to restrict access to port 27017 only from trusted IP addresses.
