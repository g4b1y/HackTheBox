```bash
nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -p- -oN "/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/_full_tcp_nmap.txt" -oX "/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/xml/_full_tcp_nmap.xml" 10.129.234.81
```

[/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/_full_tcp_nmap.txt](file:///home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/_full_tcp_nmap.txt):

```
# Nmap 7.98 scan initiated Wed Mar 18 23:29:08 2026 as: /usr/lib/nmap/nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -p- -oN /home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/_full_tcp_nmap.txt -oX /home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/xml/_full_tcp_nmap.xml 10.129.234.81
adjust_timeouts2: packet supposedly had rtt of -229702 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -229702 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -813105 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -813105 microseconds.  Ignoring time.
Nmap scan report for forgotten.htb (10.129.234.81)
Host is up, received user-set (0.026s latency).
Scanned at 2026-03-18 23:29:09 CET for 52s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 28:c7:f1:96:f9:53:64:11:f8:70:55:68:0b:e5:3c:22 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMIbLmW6I3vlf8QRrAaFLhH3Ao7CFIvqPPmQG0Z14i0SlPfX9IZobRkjLOB0ncKb5oQ/0SXLnU60rnUe+7Xe6BU=
|   256 02:43:d2:ba:4e:87:de:77:72:ce:5a:fa:86:5c:0d:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICGL/2c6HVh+6F9RbNsZpoYJ2jv4C8SGqtskv0GGuU2P
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.56
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: 403 Forbidden
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.14
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=3/18%OT=22%CT=1%CU=40516%PV=Y%DS=2%DC=T%G=Y%TM=69BB276
OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=FC%GCD=1%ISR=100%TI=Z%CI=Z%TS=A)OPS(O1=M5
OS:52ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST11NW7%O
OS:6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%D
OS:F=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0
OS:%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=
OS:Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%
OS:RD=0%Q=)T7(R=N)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G
OS:%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 22.262 days (since Tue Feb 24 17:12:11 2026)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=252 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: Host: 172.17.0.2; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   27.67 ms 10.10.14.1
2   27.83 ms forgotten.htb (10.129.234.81)

Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Mar 18 23:30:01 2026 -- 1 IP address (1 host up) scanned in 53.05 seconds

```
