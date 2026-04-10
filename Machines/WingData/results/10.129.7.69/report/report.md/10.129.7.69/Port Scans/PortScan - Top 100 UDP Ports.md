```bash
nmap -vv --reason -Pn -T4 -sU -A --top-ports 100 -oN "/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_top_100_udp_nmap.txt" -oX "/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/xml/_top_100_udp_nmap.xml" 10.129.7.69
```

[/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_top_100_udp_nmap.txt](file:///home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_top_100_udp_nmap.txt):

```
# Nmap 7.98 scan initiated Fri Mar 20 20:48:53 2026 as: /usr/lib/nmap/nmap -vv --reason -Pn -T4 -sU -A --top-ports 100 -oN /home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/_top_100_udp_nmap.txt -oX /home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/xml/_top_100_udp_nmap.xml 10.129.7.69
adjust_timeouts2: packet supposedly had rtt of -103073 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -103073 microseconds.  Ignoring time.
Nmap scan report for wingdata.htb (10.129.7.69)
Host is up, received user-set (0.025s latency).
Scanned at 2026-03-20 20:48:53 CET for 1117s
All 100 scanned ports on wingdata.htb (10.129.7.69) are in ignored states.
Not shown: 100 open|filtered udp ports (no-response)
Too many fingerprints match this host to give specific OS details
TCP/IP fingerprint:
SCAN(V=7.98%E=4%D=3/20%OT=%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=69BDA902%P=x86_64-pc-linux-gnu)
SEQ()
SEQ(II=I)
U1(R=N)
IE(R=Y%DFI=N%TG=40%CD=S)

Network Distance: 2 hops

TRACEROUTE (using proto 1/icmp)
HOP RTT      ADDRESS
1   25.25 ms 10.10.14.1
2   28.38 ms wingdata.htb (10.129.7.69)

Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar 20 21:07:30 2026 -- 1 IP address (1 host up) scanned in 1116.66 seconds

```
