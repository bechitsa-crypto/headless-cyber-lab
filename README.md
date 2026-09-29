# headless-cyber-lab
Isolated active-directory/virtualization home lab detailing vulnerability analysis, service exploitation (vsftpd 2.3.4, Samba, UnrealIRCd), and defensive mitigations.


# Hands-On Cybersecurity Lab: Metasploitable 2 Vulnerability Analysis

## Overview
This repository documents the setup, execution, and analysis of a host-only virtualized cybersecurity lab. The project demonstrates network service discovery, vulnerability identification, controlled exploitation, and defensive remediation strategies in an isolated environment.

---

## Lab Architecture & Environment Setup

* **Host System:** Windows 10 (Headless configuration managed via Parsec remote access).
* **Hypervisor:** VMware Workstation Pro.
* **Network Topology:** Isolated Host-Only Network (`VMnet1`). Traffic is strictly confined to prevent external network exposure.
* **Attacker Machine:** Kali Linux (`192.168.48.128`).
* **Target Machine:** Metasploitable 2 (`192.168.48.129`).

* [ Kali Linux (192.168.214.129) ] <--- Host-Only (VMnet1) ---> [ Metasploitable 2 (192.168.214.128) ]
|
(Isolated / No Internet)

## Service Reconnaissance & Discovery

A full service version scan was executed from the attacker host using Nmap:

```bash
nmap -sV 192.168.214.128

Key Findings:

Port 21/TCP: vsftpd 2.3.4 (Known backdoor vulnerability)

Port 139/445/TCP: Samba smbd 3.X - 4.X (Command injection via username map script)

Port 6667/TCP: UnrealIRCd (Trojaned source distribution)




Vulnerability Deep-Dive & Exploitation Analysis


1. VSFTPD 2.3.4 Backdoor (CVE-2011-2523)
Mechanism: In July 2011, the vsftpd-2.3.4.tar.gz distribution archive was compromised. A malicious trigger was inserted into user authentication routines (str.c).

Trigger: Supplying a username ending with :) forces the service to bind a root shell listener to TCP port 6200.

Execution: Successfully obtained root remote access using Metasploit (exploit/unix/ftp/vsftpd_234_backdoor) and verified privilege levels (getuid / whoami).

2. UnrealIRCd 3.2.8.1 Backdoor (CVE-2010-2075)
Mechanism: Trojaned source archive containing a malicious execution string inside the DEBUG3_DOLOG_SYSTEM macro.

Trigger: Raw TCP packet sequences prepended with AB; pass directly to standard OS system execution functions.

3. Samba username map script (CVE-2007-2447)
Mechanism: Inadequate input validation on MS-RPC requests allowed unescaped shell metacharacters (|, ;, `) in the username field.

Trigger: Input passed to /bin/sh without sanitization results in arbitrary system command execution as root.

Mitigation & Remediation Strategies
To secure these services in a enterprise setting:

- Software Updates: Upgrade vsftpd to version 2.3.5 or later and Samba to version 3.0.25a or later.

- Supply Chain Integrity: Verify cryptographic hashes (SHA-256) and GPG signatures of downloaded source archives prior to deployment.

- Network Segmentation: Implement strict firewall rules (iptables/UFW) to block external access to management and file-sharing ports (139, 445, 6200).

- Least Privilege: Ensure service daemons run under non-root, low-privilege service accounts.
