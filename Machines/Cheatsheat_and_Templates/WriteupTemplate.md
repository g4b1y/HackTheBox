# [Machine Name] - HackTheBox Writeup

**Date:** {DATE}
**OS:** {OS}
**IP Address:** {IP}
**Difficulty:** {DIFFICULTY}
**Points:** {POINTS}

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **[Machine Name]**. 

*   **Initial Access:** [Brief description of how access was gained]
*   **Privilege Escalation:** [Brief description of how root/system was reached]
*   **Key Learning Points:** 
    * [Point 1]
    * [Point 2]

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| * | SSH | [Version] | [Notes] |
| * | HTTP | [Version] | [Notes] |

**Nmap Command:**
```bash
nmap -sC -sV -oN nmap/initial [IP]
```

## 2.2. Web Enumeration
**Strategy:** [e.g., Directory Brute Forcing, Subdomain Enumeration]

### Directory Brute Forcing
```bash
gobuster dir -u http://[IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Virtual Host Discovery
```bash
ffuf -u http://[IP] -H "Host: FUZZ.[DOMAIN]" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** [e.g., SQL Injection, RCE]
*   **Vector:** [e.g., Login Form, vulnerable API endpoint]

## 3.2. Exploitation Path
[Detailed steps to obtain the user flag]

**Payload/Tool:**
```bash
[Payload or tool command]
```

**User Flag:**
```bash
cat user.txt
# [HASH]
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
[Commands used like `linpeas.sh`, `sudo -l`, etc.]

## 4.2. Exploitation Path
[Detailed steps to obtain the root flag]

**Vulnerability:** [e.g., SUID Binary, Kernel Exploit]
**Payload/Tool:**
```bash
[Payload or tool command]
```

**Root Flag:**
```bash
cat /root/root.txt
# [HASH]
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| [User] | [Pass] | [e.g., config.php] |

---

# 6. Recommendations & Mitigation
1. **[Issue 1]:** [Mitigation strategy]
2. **[Issue 2]:** [Mitigation strategy]


