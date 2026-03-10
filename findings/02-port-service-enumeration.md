# 02 – Open Port & Service Enumeration

## Objective
Identify all open ports, running services, and software versions on the target host `10.6.6.23`.

---

## Step 1: Default Port Scan

```bash
nmap 10.6.6.23
```

By default, Nmap performs a TCP connect scan (`-sT`) of the 1,000 most common ports.

**Result:** 6 ports identified as open.

| Port | State | Service |
|---|---|---|
| 21/tcp | open | ftp |
| 80/tcp | open | http |
| 139/tcp | open | netbios-ssn |
| 445/tcp | open | microsoft-ds |
| (additional ports) | open | (see scan output) |

---

## Step 2: OS Detection

```bash
sudo nmap -O 10.6.6.23
```

**Result:** Target is running a **Linux operating system**.

OS detection requires root privileges as it analyses subtle differences in TCP/IP stack behaviour to fingerprint the OS.

---

## Step 3: Service Version Detection

```bash
nmap -v -p21 -sV -T4 10.6.6.23
```

| Flag | Purpose |
|---|---|
| `-v` | Verbose output |
| `-p21` | Target port 21 (FTP) specifically |
| `-sV` | Detect service name and version |
| `-T4` | Fast timing (aggressive but detectable) |

**Result:** FTP service confirmed as **vsftpd 3.0.3** — running and open.

---

## Step 4: Aggressive Scan

```bash
nmap -A -p21 10.6.6.23
```

`-A` combines OS detection, version detection, script scanning, and traceroute in a single command. It is intrusive and will be detected by most IDS systems — only use with explicit authorisation.

**Key output from aggressive scan:**
```
21/tcp  open  ftp  vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r-- file1.txt
| -rw-r--r-- file2.txt
| -rw-r--r-- file3.txt
|_ -rw-r--r-- supersecretfile.txt
```

4 files are accessible via anonymous FTP login — including one named `supersecretfile.txt`.

---

## Evidence

> nmap default scan output showing open ports_
>
> <img width="423" height="264" alt="image" src="https://github.com/user-attachments/assets/a040c36b-62af-457d-9b4c-623dce4a6dff" />

> nmap -A output showing vsftpd and anonymous FTP files_
>
> <img width="440" height="275" alt="image" src="https://github.com/user-attachments/assets/1cb3aee1-1a52-4574-90ce-bc5501963037" />


---

## TCP Port Status Reference

| Status | Response Received | Interpretation |
|---|---|---|
| Open | TCP SYN-ACK | A service is actively listening on this port |
| Closed | TCP RST | No service is listening |
| Filtered | No response / ICMP unreachable | Port is being blocked by a firewall |
