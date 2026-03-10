# 🔍 Network Audit using Nmap — Active Reconnaissance Lab

> **Coursework lab for FIC 4040 – Information Technology & Cybercrime | USIU-Africa**  
> A structured network audit and active reconnaissance exercise conducted on a DMZ network using Nmap on Kali Linux, simulating a real-world incident response scenario.

---

## 🎯 Scenario

A Wireshark capture flagged unusual activity on a machine at **10.6.6.23** in the DMZ network (`10.6.6.0/24`). As the analyst, the objective was to perform active reconnaissance to identify running services, open ports, OS details, and potential security vulnerabilities — then produce a risk report with remediation recommendations.

---

## 🛠️ Tools & Environment

| Tool | Purpose |
|---|---|
| **Nmap 7.93** | Network scanning, port discovery, service/OS detection |
| **Nmap Scripting Engine (NSE)** | SMB enumeration, user/share discovery |
| **Kali Linux** | Attack/analyst platform on the 10.6.6.0/24 network |

---

## 📋 Findings Summary

| # | Finding | Severity | Status |
|---|---|---|---|
| 01 | Host Discovery — DMZ Network Sweep | Info | ✅ Completed |
| 02 | Open Port & Service Enumeration | High | ✅ Completed |
| 03 | Anonymous FTP Access + File Exposure | Critical | ✅ Confirmed |
| 04 | SMB Misconfiguration & Anonymous Share Access | Critical | ✅ Confirmed |
| 05 | OS & Version Detection | Info | ✅ Completed |

---

## 📁 Findings

→ [01 – Host Discovery](findings/01-host-discovery.md)  
→ [02 – Port & Service Enumeration](findings/02-port-service-enumeration.md)  
→ [03 – Anonymous FTP Access](findings/03-anonymous-ftp.md)  
→ [04 – SMB Misconfiguration](findings/04-smb-misconfiguration.md)  
→ [05 – Security Risk Report & Recommendations](findings/05-risk-report.md)  

---

## 🔑 Key Nmap Commands Used

```bash
# Host discovery — identify live hosts on the DMZ
nmap -sn 10.6.6.0/24

# Default TCP scan — identify open ports
nmap 10.6.6.23

# OS detection
sudo nmap -O 10.6.6.23

# Service version detection on FTP (verbose, fast timing)
nmap -v -p21 -sV -T4 10.6.6.23

# Aggressive scan — OS + version + scripts + traceroute
nmap -A -p21 10.6.6.23

# SMB service enumeration
nmap -A -p139,445 10.6.6.23

# NSE: Enumerate SMB users
nmap --script smb-enum-users.nse -p139,445 10.6.6.23

# NSE: Enumerate SMB shares
nmap --script smb-enum-shares.nse -p445 10.6.6.23
```

---

## 🧠 Key Takeaways

- **Anonymous FTP access** is a critical misconfiguration — anyone on the network can read, download, and potentially upload files without credentials
- **SMB anonymous READ/WRITE access** to shares (including one labelled "Confidential Workfiles") represents a severe data exposure risk
- **Nmap's NSE scripts** dramatically extend recon capability beyond basic port scanning — user enumeration, share discovery, and vulnerability checks are all possible with built-in scripts
- The same tools used for legitimate network auditing are used by attackers for reconnaissance — the difference is authorisation and intent
- **Defence-in-depth** is essential: firewalls, network segmentation, patch management, and MFA together reduce the attack surface that tools like Nmap can exploit

---

## ⚠️ Disclaimer

This lab was conducted in a **controlled educational environment** using a simulated DMZ network. All techniques documented here are for **ethical and educational purposes only**. Do not apply these techniques against systems you do not own or have explicit written permission to test.
