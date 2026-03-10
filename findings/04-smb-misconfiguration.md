<img width="461" height="288" alt="image" src="https://github.com/user-attachments/assets/2650312a-a7c0-447b-af70-781fb2decf8e" /># 04 – SMB Misconfiguration & Anonymous Share Access

## Vulnerability
**SMB Anonymous READ/WRITE Access** — SMB shares, including one labelled "Confidential Workfiles", are accessible without authentication. User enumeration is also possible via NSE scripts.

**Severity:** Critical  
**Affected Ports:** 139/tcp (NetBIOS), 445/tcp (SMB)

---

## Step 1: SMB Service Enumeration

```bash
nmap -A -p139,445 10.6.6.23
```

**Key findings:**
```
139/tcp  open  netbios-ssn  Samba smbd 3.X-4.X
445/tcp  open  netbios-ssn  Samba smbd 4.9.5-Debian
| smb-security-mode:
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

- Target is running **Samba 4.9.5 on Debian Linux**
- **SMB message signing is disabled** — making the server vulnerable to man-in-the-middle (MITM) relay attacks
- Server is a member of the default **WORKGROUP**

---

## Step 2: User Enumeration via NSE

```bash
nmap --script smb-enum-users.nse -p139,445 10.6.6.23
```

**Result:** 2 SMB usernames discovered:
- `Arbiter`
- `MasterChief`

Enumerating valid usernames without credentials gives an attacker a direct target list for brute-force or credential stuffing attacks.

---

## Step 3: Share Enumeration via NSE

```bash
nmap --script smb-enum-shares.nse -p445 10.6.6.23
```

**Result:**

| Share | Type | Anonymous Access |
|---|---|---|
| `IPC$` | Hidden (IPC) | READ/WRITE |
| `print$` | Disk (Printer Drivers) | READ/WRITE |
| `workfiles` | Disk ("Confidential Workfiles") | READ/WRITE |

**All three shares allow anonymous READ/WRITE access** — including the share explicitly labelled "Confidential Workfiles". An unauthenticated attacker can read, modify, or delete files in these directories.

---

## Evidence

> smb-enum-users output showing Arbiter and MasterChief_
>
> <img width="489" height="306" alt="image" src="https://github.com/user-attachments/assets/4ca40536-3744-4f1a-9481-6a0f98341f64" />

> _Add screenshot: smb-enum-shares output showing anonymous READ/WRITE access_
>
> <img width="461" height="288" alt="image" src="https://github.com/user-attachments/assets/8d1e6fa9-1ec7-47a0-9039-c96ce9ec68ea" />


---

## Impact

- **Confidential data exposed** to any unauthenticated user on the network
- **User accounts enumerated** without credentials — direct input for brute-force attacks
- **SMB message signing disabled** — enables NTLM relay attacks where an attacker intercepts and replays authentication to gain access to other systems
- If SMBv1 is active, the server may be vulnerable to **EternalBlue (MS17-010)** — a critical exploit used in WannaCry ransomware that enables remote code execution

---

## Remediation

- **Disable anonymous SMB access** — require authentication on all shares
- Apply **least privilege** to share permissions — users should only access what their role requires
- **Enable SMB message signing** to prevent relay attacks
- Disable **SMBv1** entirely — it is deprecated and critically vulnerable
- Restrict SMB ports (139, 445) at the firewall — they should not be accessible from untrusted network segments
- Rename shares to avoid revealing the nature of their contents (e.g., "Confidential Workfiles")
- Enable SMB audit logging to monitor access and detect anomalies
