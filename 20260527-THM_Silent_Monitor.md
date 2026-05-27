# THM — Silent Monitor

| Field | Value |
| --- | --- |
| Engagement | THM_Silent Monitor |
| Target IP | 10.145.167.168 |
| System | Linux |
| URL / host | http://10.145.167.168:5050 |
| Export date | May 27, 2026 |
| Progress | 5 / 99 tasks completed |

---

## Engagement Notes

---

### `[05/27/26 10:43 AM]` — Start

The room description is as follows

```
Green Lights, Dark Corners

CorpNet's internal network operations centre has been running quietly for years. Monitoring hosts, logging events, and keeping the infrastructure alive. Or so it seems. A tip from a disgruntled contractor suggests that someone on the NOC team has been cutting corners, leaving doors open, and hiding things in places no one thinks to look.

The portal is up. The services show green. The audit log looks clean.

But clean logs can be written by anyone.

Your job is to get in, move through the system, and find out what is really running behind the secret dashboard.
```

---

### `[05/27/26 10:43 AM]`

START!

---

- ✅ Port scanning (top 1000)

### `[05/27/26 10:44 AM]` 📌
  > ```
  > PORT     STATE SERVICE
  > 22/tcp   open  ssh
  > 5050/tcp open  mmcc
  > ```

---

### `[05/27/26 10:46 AM]`
  > ```
  > Not shown: 998 closed tcp ports (reset)
  > PORT     STATE SERVICE VERSION
  > 22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
  > 5050/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.10.12)
  > ```



---

### `[05/27/26 10:47 AM]`

It looks like port 5050 is http, lets see if it's reachable from a browser

---

### `[05/27/26 10:48 AM]`

there appears to be a basic website at http://10.145.167.168:5050

---

### `[05/27/26 10:51 AM]`

Looking at the inspector and view page source, nothing is standing out to me.

---

### `[05/27/26 10:53 AM]`
  > Nothing additional found

---

### `[05/27/26 10:58 AM]`

Searching for directories with

```
gobuster dir -u http://10.145.167.168:5050 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

---

### `[05/27/26 10:59 AM]`

Signin portal found at

```
http://10.145.167.168:5050/internal
```

---

### `[05/27/26 11:04 AM]`

I don't have any other info at this point, so I'll see if I can brute force with Hydra

---

### `[05/27/26 11:07 AM]`

capturing a failed login through Burp I see the failed login produces "Invalid username or password."

---

### `[05/27/26 11:18 AM]`

testing hydra with

```
hydra -L /usr/share/wordlists/usersshort.txt-P /usr/share/wordlists/rockyou.txt 10.145.167.168 http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid username or password."
```

---

### `[05/27/26 11:20 AM]`

Updated hydra syntax

```
hydra -L /usr/share/wordlists/usersshort.txt -P /usr/share/wordlists/rockyou.txt 10.145.167.168:5050/internal http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid username or password."
```

---

### `[05/27/26 11:23 AM]`

I'm getting an interesting error from hydra

```
Error: network size may only be between /16 and /31: 10.145.167.168:5050/internal
```

---

### `[05/27/26 11:30 AM]`

I finally got the syntax right with

```
hydra -L /usr/share/wordlists/usersshort.txt -P /usr/share/wordlists/rockyou.txt 10.145.167.168 -s 5050 http-post-form "/internal:username=^USER^&password=^PASS^:F=Invalid username or password."
```

---

### `[05/27/26 11:37 AM]`

Hydra isn't finding much, perhaps I can use it to BF SSH

---

 ### `[05/27/26 11:37 AM]`
  > `/internal`

---

### `[05/27/26 11:38 AM]`

The NOC portal is v2.4.1, but there is no reference as to what the software is? Perhaps I can enum that with a tool?

---
### `[05/27/26 11:40 AM]`
  > ```
  > ┌──(kali㉿kali)-[~/tmp]
  > └─$ whatweb -a 3 http://10.145.167.168:5050
  > http://10.145.167.168:5050 [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[Werkzeug/2.0.2 Python/3.10.12], IP[10.145.167.168], Python[3.10.12], Title[CorpNet — Network Operations Centre], Werkzeug[2.0.2]
  > ```

  ---

### `[05/27/26 11:48 AM]`

Trying to find clues, I went to look at the page again at http://10.145.167.168:5050 for links. I'm not visually seeing anything so I used a python script to search for links

```
  GNU nano 8.7.1                                      FindLinksOnWebpage.py                                                
# First install the line below on CLI
#pip3 install requests beautifulsoup4 

from bs4 import BeautifulSoup
import requests 

html = requests.get('http://10.145.167.168:5050').text
soup = BeautifulSoup(html, 'lxml')
links = soup.find_all('a', href=True)

for i in links:
 print(i['href'])
```

No results, I was not missing anything.

---

### `[05/27/26 11:51 AM]`

The page says there are 14 hosts, perhaps there are other VMs spun up on the network?
Searching wtth

```
nmap 10.145.167.0/24
```

---

### `[05/27/26 11:52 AM]`

I found another host

```
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-27 11:45 -0700
Nmap scan report for 10.145.167.168
Host is up (0.067s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
5050/tcp open  mmcc

Nmap scan report for 10.145.167.227
Host is up (0.070s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy

Nmap done: 256 IP addresses (2 hosts up) scanned in 35.42 seconds
```

---

### `[05/27/26 11:54 AM]`

There's a live site at http://10.145.167.227/

---

### `[05/27/26 11:54 AM]`

and http://10.145.167.227:8080 is a default Apache2 page

---

### `[05/27/26 12:08 PM]`

http://10.145.167.227:3306/ gave me a response `f���ÿjHost 'ip-192-168-137-113.us-west-2.compute.internal' is not allowed to connect to this MySQL server`

---

### `[05/27/26 12:10 PM]`

Creating an account at 10.145.167.227/register.php

```
Name gnope
email none@no.com
pass 123456
```

---

### `[05/27/26 12:10 PM]`

a registered URL of http://10.145.167.227/login.php?registered=1 seems promising.

---

### `[05/27/26 12:12 PM]`

Applying for a job gives another URL of http://10.145.167.227/dashboard.php?apply=6
Let's see if this is exploitable somehow.

---

### `[05/27/26 12:13 PM]`

1-10 cycles through the jobs, 11 does not lead anywhere interesting

---

### `[05/27/26 12:19 PM]`

Looking through the jobs, I see some names

```
Priya Desai
James Crawford
```

I'm also seeing some possible useful info on platforms used.

```
Kubernetes
Terraform
AWS
Prometheus
Grafana
ELK stack
Tableau
Power BI
```

---

### `[05/27/26 12:20 PM]`

I also get the URL `http://10.145.167.227/profile.php?id=7` when I click on my profile. Maybe I can gain access to other profiles from here.

---

### `[05/27/26 12:21 PM]`

We're on to something. `http://10.145.167.227/profile.php?id=1` is

```
Sarah Mitchell
Administrator
s.mitchell@recruitx.thm
```

---

### `[05/27/26 12:25 PM]`

and suddenly it appears that my host at 10.145.167.227 is down. No pages are loading and I'm not able to ping it. I'm still able to ping 10.145.167.168 though. I have some more info, let's see if I can get anywhere with it.

---

### `[05/27/26 12:31 PM]`

with the new info, I'll try hydra again with a new user list I made based on users and email I found.

```
hydra -L /home/kali/tmp/user.txt -P /usr/share/wordlists/rockyou.txt 10.145.167.168 -s 5050 http-post-form "/internal:username=^USER^&password=^PASS^:F=Invalid username or password."
```

---

### `[05/27/26 12:36 PM]`

While Hydra is running, I'm googling and seeing that Grafana v2.4.1 has a possible exploit to read files. This might be what I'm looking for.

---

### `[05/27/26 12:41 PM]`

running an exploit through Metasploit with the following parameters

```
Module options (auxiliary/admin/http/grafana_auth_bypass):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   COOKIE                      no        Decrypt captured cookie
   RHOSTS     10.145.167.168   yes       Address of target
   RPORT      5050             yes       Port of target
   SSL        false            yes       set SSL/TLS based connection
   TARGETURI  /                no        Base URL of grafana instance
   THREADS    1                yes       The number of concurrent threads (max one per host)
   USERNAME                    no        Valid username
   VERSION    2-4              yes       Grafana version: "2-4" or "5" (Accepted: 2-4, 5)
```

It finds a potential auth exploit in the form of a cookie

```
Encrypted remember cookie: 2e297dbc277785cb563836f444add45002b1964a21baed305b50284b
```

---

### `[05/27/26 12:46 PM]`

I changed the TARGETURI to /internal and got the cookie `713e2461d1ffd97c96bf1918ea8c34f6b20fcf85a382f93b3f0a5da2`, still no luck

---

### `[05/27/26 12:47 PM]`

for reference, I'm injecting the found cookies through Burp

---

### `[05/27/26 12:56 PM]`

I don't think I ever got around to running Hydra on SSH. Perhaps I'll try that.

---

### `[05/27/26 12:58 PM]`

```
hydra -L /home/kali/tmp/user.txt -P /usr/share/wordlists/rockyou.txt ssh://10.145.167.168
```

---

### `[05/27/26 01:10 PM]`

Another nmap scan shows multiple machines up and running. I'm thinking this is not the right path and these other machines are spun up by other users. I've probably been chasing a mguffin for a while.

```
┌──(kali㉿kali)-[~]
└─$ nmap 10.145.167.0/24                                                   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-27 13:02 -0700
Nmap scan report for 10.145.167.67
Host is up (0.070s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap scan report for 10.145.167.111
Host is up (0.070s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp

Nmap scan report for 10.145.167.168
Host is up (0.070s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
5050/tcp open  mmcc

Nmap scan report for 10.145.167.202
Host is up (0.071s latency).
Not shown: 992 closed tcp ports (reset)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
110/tcp open  pop3
143/tcp open  imap
993/tcp open  imaps
995/tcp open  pop3s
```

---

### `[05/27/26 01:29 PM]`

I have no idea on how to procede. I feel like everything I've done has been a dead end. I know I'm missing something stupid simple here. I'll have to come back to this another day.

---

### `[05/27/26 01:34 PM]`

a last minute nmap scan. perhaps some keys to exploit after I eat...

```
┌──(kali㉿kali)-[~/tmp]
└─$ nmap -sC 10.145.167.168 -p-
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-27 13:27 -0700
Nmap scan report for 10.145.167.168
Host is up (0.068s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
| ssh-hostkey: 
|   256 a1:93:b6:da:bf:54:05:b1:c0:a3:47:00:fb:3c:72:fe (ECDSA)
|_  256 7a:2f:e7:d7:39:8d:a0:2d:96:5a:a4:c5:f4:c9:a4:34 (ED25519)
5050/tcp open  mmcc
```

---

### `[05/27/26 01:59 PM]`

A google search leads me to look into a vuln for OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0). The room description says something about looking where you don't think to look. I didn't think to look here. Let's see if we can get anywhere with this.

Looking into the vuln, it seems like there's a way to verify with any private key, if I'm reading this right. that would be an easy way in. Fingers crossed..

---

### `[05/27/26 02:27 PM]`

I generated a random RSA private key and put it into a id_rsa file in a temp folder.

```
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAhoV16uYbm3NlzkXZYHzBYWZUg9VJnXAuDw+nVEoJlLSeQjsA
[...key omitted for brevity...]
-----END RSA PRIVATE KEY-----
```

---

### `[05/27/26 02:27 PM]`

next I need to limit perms for the file

```
chmod 600 id_rsa
```

---

### `[05/27/26 02:28 PM]`

and I'll try to login with

```
ssh -i id_rsa root@10.145.167.168
```

---

### `[05/27/26 02:30 PM]`

I didn't get any backlash for the private key, but it's still asking for a password. I'll try with admin instead of root.

---

### `[05/27/26 02:59 PM]`

I have no clue.... nothing is working... for a medium room that's rated for 30 minutes, I've spent way too long trying to figure this out. I'll need to come back to this another time...
