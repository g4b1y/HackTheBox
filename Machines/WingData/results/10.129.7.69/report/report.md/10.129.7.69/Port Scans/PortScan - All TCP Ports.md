```bash
nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -p- -oN "/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_full_tcp_nmap.txt" -oX "/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/xml/_full_tcp_nmap.xml" 10.129.7.69
```

[/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_full_tcp_nmap.txt](file:///home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_full_tcp_nmap.txt):

```
# Nmap 7.98 scan initiated Fri Mar 20 20:48:53 2026 as: /usr/lib/nmap/nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -p- -oN /home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_full_tcp_nmap.txt -oX /home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/xml/_full_tcp_nmap.xml 10.129.7.69
adjust_timeouts2: packet supposedly had rtt of 9704383 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of 9704383 microseconds.  Ignoring time.
Nmap scan report for wingdata.htb (10.129.7.69)
Host is up, received user-set (0.049s latency).
Scanned at 2026-03-20 20:48:53 CET for 1333s
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 a1:fa:95:8b:d7:56:03:85:e4:45:c9:c7:1e:ba:28:3b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBL+8LZAmzRfTy+4t8PJxEvRWhPho8aZj9ImxRfWn9TKepkxh8pAF3WDu55pd/gaSUGIo9cuOvv+3r6w7IuCpqI4=
|   256 9c:ba:21:1a:97:2f:3a:64:73:c1:4c:1d:ce:65:7a:2f (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFFmcxflCAAe4LPgkg7hOxJen41bu6zaE/y08UnA4oRp
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.66
|_http-server-header: Apache/2.4.66 (Debian)
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-title: WingData Solutions
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%), Linux 5.4 - 5.10 (90%), Linux 2.6.32 - 3.5 (86%), Crestron XPanel control system (86%), Android 8 - 9 (Linux 3.18 - 4.4) (86%), Linux 2.6.32 - 3.13 (86%), Linux 3.10 - 4.11 (86%), Linux 3.13 - 4.4 (86%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.98%E=4%D=3/20%OT=22%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=69BDA9DA%P=x86_64-pc-linux-gnu)
SEQ(SP=102%GCD=9%ISR=106%TI=Z%TS=D)
SEQ(SP=107%GCD=1%ISR=108%TI=Z%II=I%TS=C)
OPS(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST11NW7%O6=M552ST11)
WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)
ECN(R=Y%DF=Y%TG=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)
T1(R=Y%DF=Y%TG=40%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=Y%TG=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
U1(R=N)
IE(R=Y%DFI=N%TG=40%CD=S)

Uptime guess: 3.701 days (since Tue Mar 17 04:21:27 2026)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   41.54 ms 10.10.14.1
2   42.00 ms wingdata.htb (10.129.7.69)

Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar 20 21:11:06 2026 -- 1 IP address (1 host up) scanned in 1333.12 seconds

```
