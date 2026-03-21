```bash
nmap -vv --reason -Pn -T4 -sV -p 22 --script="banner,ssh2-enum-algos,ssh-hostkey,ssh-auth-methods" -oN "/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp22/tcp_22_ssh_nmap.txt" -oX "/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp22/xml/tcp_22_ssh_nmap.xml" 10.129.7.69
```

[/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp22/tcp_22_ssh_nmap.txt](file:///home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp22/tcp_22_ssh_nmap.txt):

```
# Nmap 7.98 scan initiated Fri Mar 20 20:49:51 2026 as: /usr/lib/nmap/nmap -vv --reason -Pn -T4 -sV -p 22 --script=banner,ssh2-enum-algos,ssh-hostkey,ssh-auth-methods -oN /home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp22/tcp_22_ssh_nmap.txt -oX /home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp22/xml/tcp_22_ssh_nmap.xml 10.129.7.69
Nmap scan report for wingdata.htb (10.129.7.69)
Host is up, received user-set (0.026s latency).
Scanned at 2026-03-20 20:49:51 CET for 7s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh2-enum-algos: 
|   kex_algorithms: (12)
|       sntrup761x25519-sha512
|       sntrup761x25519-sha512@openssh.com
|       curve25519-sha256
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp256
|       ecdh-sha2-nistp384
|       ecdh-sha2-nistp521
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
|_    password
|_banner: SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u7
| ssh-hostkey: 
|   256 a1:fa:95:8b:d7:56:03:85:e4:45:c9:c7:1e:ba:28:3b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL+8LZAmzRfTy+4t8PJxEvRWhPho8aZj9ImxRfWn9TKepkxh8pAF3WDu55pd/gaSUGIo9cuOvv+3r6w7IuCpqI4=
|   256 9c:ba:21:1a:97:2f:3a:64:73:c1:4c:1d:ce:65:7a:2f (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFFmcxflCAAe4LPgkg7hOxJen41bu6zaE/y08UnA4oRp
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar 20 20:49:58 2026 -- 1 IP address (1 host up) scanned in 7.04 seconds

```
