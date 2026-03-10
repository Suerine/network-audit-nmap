# 03 – Anonymous FTP Access & File Exposure

## Vulnerability
**Anonymous FTP Login Enabled** — the FTP server on port 21 accepts connections without credentials, exposing internal files to any host on the network.

**Severity:** Critical  
**Affected Service:** vsftpd 3.0.3 on port 21

---

## Discovery

Identified during the aggressive Nmap scan (`nmap -A -p21 10.6.6.23`). The NSE script `ftp-anon` automatically tested for anonymous login and confirmed access.

```
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--  1 0  0  16 Aug 13 2021  file1.txt
| -rw-r--r--  1 0  0  16 Aug 13 2021  file2.txt
| -rw-r--r--  1 0  0  29 Aug 13 2021  file3.txt
|_-rw-r--r--  1 0  0  26 Aug 13 2021  supersecretfile.txt
```

---

## Impact

- **4 files** are directly accessible without any authentication
- The presence of `supersecretfile.txt` strongly suggests sensitive data is stored on an openly accessible server
- Anonymous FTP typically allows both `get` (download) and `put` (upload) commands — meaning an attacker could **exfiltrate existing files or upload malicious payloads** (e.g., web shells, backdoors)
- This vulnerability requires zero credentials and can be exploited by any host on the network

---

## Evidence

> _Add screenshot: Nmap output showing anonymous FTP login and file listing_

---

## Remediation

- **Disable anonymous FTP access immediately** — require authentication for all FTP connections
- Migrate from FTP to **SFTP or FTPS** — plain FTP transmits credentials and data in cleartext, making it trivially interceptable with tools like Wireshark
- Apply **least privilege** to FTP directories — remove write access unless explicitly required
- Audit all files currently exposed via FTP and assess whether any sensitive data has already been accessed
- If FTP must remain enabled, restrict access by IP using firewall rules and enable logging on all connections
