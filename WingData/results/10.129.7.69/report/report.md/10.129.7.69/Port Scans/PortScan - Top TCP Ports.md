```bash
nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -oN "/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_quick_tcp_nmap.txt" -oX "/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/xml/_quick_tcp_nmap.xml" 10.129.7.69
```

[/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_quick_tcp_nmap.txt](file:///home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_quick_tcp_nmap.txt):

```
# Nmap 7.98 scan initiated Fri Mar 20 20:48:53 2026 as: /usr/lib/nmap/nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -oN /home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_quick_tcp_nmap.txt -oX /home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/xml/_quick_tcp_nmap.xml 10.129.7.69
adjust_timeouts2: packet supposedly had rtt of -230666 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -230666 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -222585 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -222585 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -246664 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -246664 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -249909 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -249909 microseconds.  Ignoring time.
Nmap scan report for wingdata.htb (10.129.7.69)
Host is up, received user-set (0.036s latency).
Scanned at 2026-03-20 20:48:53 CET for 58s
Not shown: 998 filtered tcp ports (no-response)
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
Device type: general purpose|router|specialized|storage-misc
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (91%), MikroTik RouterOS 7.X (89%), Crestron 2-Series (86%), HP embedded (85%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:crestron:2_series cpe:/h:hp:p2000_g3
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (89%), Linux 5.4 - 5.10 (88%), Linux 2.6.32 - 3.5 (86%), Crestron XPanel control system (86%), Linux 2.6.32 - 3.13 (86%), Linux 3.10 - 4.11 (86%), Linux 3.2 - 4.14 (86%), Linux 3.4 - 3.10 (86%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.98%E=4%D=3/20%OT=22%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=69BDA4DF%P=x86_64-pc-linux-gnu)
SEQ(SP=106%GCD=1%ISR=109%TI=Z%II=I%TS=D)
SEQ(SP=FD%GCD=1%ISR=101%TI=Z%II=I%TS=C)
OPS(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST11NW7%O6=M552ST11)
WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)
ECN(R=Y%DF=Y%TG=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)
T1(R=Y%DF=Y%TG=40%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=Y%TG=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
U1(R=N)
IE(R=Y%DFI=N%TG=40%CD=S)

Uptime guess: 6.255 days (since Sat Mar 14 14:42:28 2026)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=262 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   26.85 ms 10.10.14.1
2   26.96 ms wingdata.htb (10.129.7.69)

Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar 20 20:49:51 2026 -- 1 IP address (1 host up) scanned in 58.12 seconds

```
