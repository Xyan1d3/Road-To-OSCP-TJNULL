# Sense
>Author : Xyan1d3
>Date : 2nd July 2021
>IP : 10.10.10.60
>OS : FreeBSD
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Fri Jul  2 11:39:15 2021 as: nmap -sC -sV -v -oN nmap/sense 10.10.10.60
Nmap scan report for sense.htb (10.10.10.60)
Host is up (0.064s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE  VERSION
80/tcp  open  http     lighttpd 1.4.35
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://sense.htb/
443/tcp open  ssl/http lighttpd 1.4.35
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-favicon: Unknown favicon MD5: 082559A7867CF27ACAB7E9867A8B320F
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: lighttpd/1.4.35
|_http-title: 501
| ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Issuer: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2017-10-14T19:21:35
| Not valid after:  2023-04-06T19:21:35
| MD5:   65f8 b00f 57d2 3468 2c52 0f44 8110 c622
|_SHA-1: 4f7c 9a75 cb7f 70d3 8087 08cb 8c27 20dc 05f1 bb02
|_ssl-date: TLS randomness does not represent time
```

## Gobuster
 We could now run a `gobuster` with `php,txt` extensions.
 ```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/sense]
└──╼ # gobuster dir -u https://10.10.10.60/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php -t 50 -k
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.10.60/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2021/07/02 11:59:59 Starting gobuster in directory enumeration mode
===============================================================
/help.php             (Status: 200) [Size: 6689]
/themes               (Status: 301) [Size: 0] [--> https://10.10.10.60/themes/]
/index.php            (Status: 200) [Size: 6690]                               
/stats.php            (Status: 200) [Size: 6690]                               
/css                  (Status: 301) [Size: 0] [--> https://10.10.10.60/css/]   
/includes             (Status: 301) [Size: 0] [--> https://10.10.10.60/includes/]
/license.php          (Status: 200) [Size: 6692]                                 
/system.php           (Status: 200) [Size: 6691]                                 
/edit.php             (Status: 200) [Size: 6689]                                 
/status.php           (Status: 200) [Size: 6691]                                 
/javascript           (Status: 301) [Size: 0] [--> https://10.10.10.60/javascript/]
/changelog.txt        (Status: 200) [Size: 271]
/classes              (Status: 301) [Size: 0] [--> https://10.10.10.60/classes/]   
/exec.php             (Status: 200) [Size: 6689]                                   
/widgets              (Status: 301) [Size: 0] [--> https://10.10.10.60/widgets/]   
/graph.php            (Status: 200) [Size: 6690]                                   
/tree                 (Status: 301) [Size: 0] [--> https://10.10.10.60/tree/]      
/wizard.php           (Status: 200) [Size: 6691]                                   
/shortcuts            (Status: 301) [Size: 0] [--> https://10.10.10.60/shortcuts/] 
/pkg.php              (Status: 200) [Size: 6688]                                   
/installer            (Status: 301) [Size: 0] [--> https://10.10.10.60/installer/] 
/wizards              (Status: 301) [Size: 0] [--> https://10.10.10.60/wizards/]   
/xmlrpc.php           (Status: 200) [Size: 384]
/reboot.php           (Status: 200) [Size: 6691]                                   
/interfaces.php       (Status: 200) [Size: 6695]                                   
/csrf                 (Status: 301) [Size: 0] [--> https://10.10.10.60/csrf/]
/system-users.txt     (Status: 200) [Size: 106]
/filebrowser          (Status: 301) [Size: 0] [--> https://10.10.10.60/filebrowser/]
```
## Web Enumeration
We visit the `http://10.10.10.60` and find out that it redirects to `https://10.10.10.60`.

The SSL Certificate leaks no hostname or any vital information.
On, Visiting the site we are greeted with a `pfsense` login page.
![[Pasted image 20210702121900.png]]
We try some default credentials but , We find out those don't work.

But from the `gobuster` above we find out that we have a `system-users.txt` which we could look at.
![[Pasted image 20210702122110.png]]

We find out a pair of credentials which may work on the pfsense.
![[Pasted image 20210702123916.png]]

We use these credentials and get logged in to the pfsense.
|Username|Password|
|--|--|
|rohit|pfsense|

We get logged into the `pfsense` router.
![[Pasted image 20210702124350.png]]

It is running `pfsense 2.1.3` which is vulnerable to `CVE-2014-4688` which allows ARCE.

# Initial FootHold
We have a exploit on `searchsploit` which we could use to get a reverse shell back to our box.
![[Pasted image 20210702124913.png]]

On, Running the exploit we get a reverse shell back as the `root` user of the box.
![[Pasted image 20210702125048.png]]

## Getting user Flag
```python
# id
uid=0(root) gid=0(wheel) groups=0(wheel)
# cd /home/rohit/
# ls -l
total 8
-rw-r--r--  1 rohit  nobody  1003 Oct 14  2017 .tcshrc
-rw-r--r--  1 root   nobody    32 Oct 14  2017 user.txt
# cat user.txt;echo
8721327cc232073b40d27d9c17e7348b
```

## Getting root Flag
```python
# id
uid=0(root) gid=0(wheel) groups=0(wheel)
# cd /root
# ls -l
total 28
-rw-r--r--  1 root  wheel   724 May  1  2014 .cshrc
-rw-r--r--  1 root  wheel     0 Oct 14  2017 .first_time
-rw-r--r--  1 root  wheel   167 May  1  2014 .gitsync_merge.sample
-rw-r--r--  1 root  wheel     0 May  1  2014 .hushlogin
-rw-r--r--  1 root  wheel   229 May  1  2014 .login
-rw-r--r--  1 root  wheel     0 Oct 14  2017 .part_mount
-rw-r--r--  1 root  wheel   165 May  1  2014 .profile
-rw-r--r--  1 root  wheel   165 May  1  2014 .shrc
-rw-r--r--  1 root  wheel  1003 Oct 14  2017 .tcshrc
-rw-r--r--  1 root  wheel    33 Oct 18  2017 root.txt
# cat root.txt ;echo
d08c32a5d4f8c8b10e76eb51a69f1a86
```