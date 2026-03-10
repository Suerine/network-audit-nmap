# 01 – Host Discovery (DMZ Network Sweep)

## Objective
Identify all live hosts on the `10.6.6.0/24` DMZ network before targeting the suspicious machine at `10.6.6.23`.

## Command Used

```bash
nmap -sn 10.6.6.0/24
```

**`-sn`** performs a ping scan — sends ICMP echo requests, TCP SYN to port 443, TCP ACK to port 80, and ICMP timestamp requests. Any response confirms the host is alive. No port scanning is performed at this stage.

---

## Result

**7 active hosts** identified on the DMZ network.

The target host `10.6.6.23` responded, confirming it is live and its IP stack is functioning.

---

## Evidence

> nmap -sn output showing 7 hosts up_
>
> <img width="425" height="266" alt="image" src="https://github.com/user-attachments/assets/0f664201-edfe-40ee-ab64-a99adfba3eeb" />


---

## Notes

A discovery scan is always the first step in a structured network audit — it maps the environment before any deeper scanning begins. Running `-sn` first minimises noise and avoids triggering IDS alerts from unnecessary port scans against hosts that aren't live.
