```bash
nmap -vv --reason -Pn -T4 -sV -p 22 --script="banner,ssh2-enum-algos,ssh-hostkey,ssh-auth-methods" -oN "/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp22/tcp_22_ssh_nmap.txt" -oX "/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp22/xml/tcp_22_ssh_nmap.xml" 10.129.234.81
```

[/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp22/tcp_22_ssh_nmap.txt](file:///home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp22/tcp_22_ssh_nmap.txt):

```
# Nmap 7.98 scan initiated Wed Mar 18 23:29:26 2026 as: /usr/lib/nmap/nmap -vv --reason -Pn -T4 -sV -p 22 --script=banner,ssh2-enum-algos,ssh-hostkey,ssh-auth-methods -oN /home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp22/tcp_22_ssh_nmap.txt -oX /home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp22/xml/tcp_22_ssh_nmap.xml 10.129.234.81
Nmap scan report for forgotten.htb (10.129.234.81)
Host is up, received user-set (0.029s latency).
Scanned at 2026-03-18 23:29:26 CET for 1s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.13
| ssh2-enum-algos: 
|   kex_algorithms: (11)
|       curve25519-sha256
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp256
|       ecdh-sha2-nistp384
|       ecdh-sha2-nistp521
|       sntrup761x25519-sha512@openssh.com
|       diffie-hellman-group-exchange-sha256
|       diffie-hellman-group16-sha512
|       diffie-hellman-group18-sha512
|       diffie-hellman-group14-sha256
|       kex-strict-s-v00@openssh.com
|   server_host_key_algorithms: (4)
|       rsa-sha2-512
|       rsa-sha2-256
|       ecdsa-sha2-nistp256
|       ssh-ed25519
|   encryption_algorithms: (6)
|       chacha20-poly1305@openssh.com
|       aes128-ctr
|       aes192-ctr
|       aes256-ctr
|       aes128-gcm@openssh.com
|       aes256-gcm@openssh.com
|   mac_algorithms: (10)
|       umac-64-etm@openssh.com
|       umac-128-etm@openssh.com
|       hmac-sha2-256-etm@openssh.com
|       hmac-sha2-512-etm@openssh.com
|       hmac-sha1-etm@openssh.com
|       umac-64@openssh.com
|       umac-128@openssh.com
|       hmac-sha2-256
|       hmac-sha2-512
|       hmac-sha1
|   compression_algorithms: (2)
|       none
|_      zlib@openssh.com
| ssh-auth-methods: 
|   Supported authentication methods: 
|     publickey
|     password
|_    keyboard-interactive
| ssh-hostkey: 
|   256 28:c7:f1:96:f9:53:64:11:f8:70:55:68:0b:e5:3c:22 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMIbLmW6I3vlf8QRrAaFLhH3Ao7CFIvqPPmQG0Z14i0SlPfX9IZobRkjLOB0ncKb5oQ/0SXLnU60rnUe+7Xe6BU=
|   256 02:43:d2:ba:4e:87:de:77:72:ce:5a:fa:86:5c:0d:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICGL/2c6HVh+6F9RbNsZpoYJ2jv4C8SGqtskv0GGuU2P
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Mar 18 23:29:27 2026 -- 1 IP address (1 host up) scanned in 1.51 seconds

```
