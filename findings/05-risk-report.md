# 05 – Security Risk Report & Recommendations

## Target Host: 10.6.6.23
**Analyst:** Suerine Otieno  
**Scope:** Active reconnaissance of DMZ host `10.6.6.23` on network `10.6.6.0/24`  
**Tools Used:** Nmap 7.93, Nmap Scripting Engine (NSE), Kali Linux

---

## Executive Summary

A network audit of host `10.6.6.23` revealed multiple critical security misconfigurations that expose the organisation to significant risk of unauthorised data access, lateral movement, and potential full system compromise. The host is running several services with weak or absent authentication controls, outdated protocols, and publicly accessible sensitive files. Immediate remediation is recommended.

---

## Risk Register

| Risk | Severity | Likelihood | Priority |
|---|---|---|---|
| Anonymous FTP login — file read/write without credentials | Critical | High | P1 |
| SMB anonymous READ/WRITE on "Confidential Workfiles" share | Critical | High | P1 |
| SMB message signing disabled — relay attack possible | High | Medium | P2 |
| SMB user enumeration without credentials | High | High | P2 |
| Outdated web services — potential default credentials/misconfig | Medium | Medium | P3 |
| No apparent rate limiting or brute-force protection | High | High | P2 |

---

## Detailed Findings

### 🔴 P1 — Anonymous FTP Access (Port 21)

**Finding:** The FTP service running vsftpd 3.0.3 accepts anonymous logins (FTP code 230). Four files are directly accessible, including one named `supersecretfile.txt`.

**Risk:** Any host on the network can download sensitive files or upload malicious payloads without authentication. FTP also transmits data in cleartext — credentials and file contents are interceptable via packet capture.

**Recommendation:**
- Disable anonymous FTP access immediately
- Migrate to SFTP or FTPS
- Audit current file exposure and assess breach scope
- Restrict FTP directory permissions and apply IP-based access controls

---

### 🔴 P1 — SMB Anonymous Share Access (Ports 139, 445)

**Finding:** All SMB shares — including one explicitly labelled "Confidential Workfiles" — allow anonymous READ/WRITE access. Two valid usernames (`Arbiter`, `MasterChief`) were enumerated without credentials.

**Risk:** Unauthenticated access to confidential files. Enumerated usernames provide a direct attack surface for brute-force or credential stuffing. If SMBv1 is active, the server may be vulnerable to EternalBlue (MS17-010) — enabling remote code execution.

**Recommendation:**
- Disable anonymous SMB access across all shares
- Enable SMB message signing
- Disable SMBv1 entirely
- Restrict SMB ports at the firewall to trusted hosts only
- Implement account lockout to prevent brute-force attacks against discovered usernames

---

### 🟠 P2 — SMB Message Signing Disabled

**Finding:** Nmap SMB security scan confirmed: `message_signing: disabled (dangerous, but default)`.

**Risk:** Without message signing, an attacker can intercept SMB authentication exchanges and relay them to other systems (NTLM relay attack) — potentially gaining access to additional hosts on the network without ever cracking a password.

**Recommendation:**
- Enable SMB message signing via Group Policy or Samba configuration
- Deploy network monitoring to detect relay attack patterns

---

### 🟠 P2 — Web Services with Potential Misconfigurations (Port 80/443)

**Finding:** HTTP/HTTPS services detected. Default web applications may be running with directory listing enabled, potentially exposing sensitive files.

**Risk:** Outdated web servers are vulnerable to known exploits. Default credentials and directory traversal vulnerabilities are common in misconfigured web services.

**Recommendation:**
- Identify and update all web server software
- Disable directory listing
- Remove default applications and credentials
- Run a web application vulnerability scan (Nikto, OWASP ZAP)

---

## Network-Level Recommendations

### 1. Firewall & Network Segmentation
- Implement strict firewall rules — only permit necessary traffic to/from the DMZ
- Segment the network into zones:
  - **DMZ** — public-facing services only (web, mail)
  - **Internal** — application and database servers
  - **Restricted** — critical infrastructure (Active Directory, backups)
- Block SMB ports (139, 445) from being accessible outside the internal network

### 2. Intrusion Detection & Prevention
- Deploy an IDS/IPS (Snort, Suricata, or Zeek) for real-time threat detection
- Configure alerts for port scans, brute-force attempts, and anonymous login events
- Review Nmap scan signatures — the scans performed in this audit would be detectable by a properly configured IDS

### 3. Identity & Access Management
- Enforce strong password policies across all services
- Implement multi-factor authentication (MFA) for all privileged accounts
- Disable password-based SSH — enforce key-based authentication only
- Conduct quarterly access control reviews

### 4. Patch Management
- Enable automatic security updates on all servers
- Run monthly vulnerability scans using Nessus or OpenVAS
- Immediately disable deprecated protocols: FTP (replace with SFTP), SMBv1, Telnet

---

## Conclusion

Host `10.6.6.23` presents a significant and immediate security risk to the organisation. The combination of anonymous FTP and SMB access, disabled message signing, and enumerable user accounts creates multiple pathways for an attacker to gain unauthorised access, exfiltrate data, or move laterally across the network. The two P1 findings should be treated as urgent and remediated before the next business day.
