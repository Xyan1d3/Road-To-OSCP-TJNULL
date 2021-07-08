# Popcorn
>Author : Xyan1d3
>Date : 8th July 2021
>IP : 10.10.10.6
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jul  7 22:38:48 2021 as: nmap -sC -sV -v -oN nmap/popcorn 10.10.10.6
Nmap scan report for popcorn.htb (10.10.10.6)
Host is up (0.066s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/popcorn]
└──╼ # gobuster dir -u http://10.10.10.6/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out -t 100 -x php,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.6/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2021/07/07 22:45:44 Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 177]
/test                 (Status: 200) [Size: 47034]
/test.php             (Status: 200) [Size: 47046]
/torrent              (Status: 301) [Size: 310] [--> http://10.10.10.6/torrent/]
```

## Web Enumeration
We visit the `http://10.10.10.6/torrent` and find out that we have a torrent hoster page.
![[Pasted image 20210708011534.png]]

We visit the login page and attempt to do a simple SQLi attempt.
![[Pasted image 20210708013240.png]]

We get a big fat SQL error which is a good sign as we can bypass it and login to the server.
![[Pasted image 20210708013313.png]]

We use this payload to bypass the login and get logged into the panel as `admin`.
![[Pasted image 20210708013906.png]]

# Initial FootHold as www-data
The exploit is to change torrent image and do some basic extension bypass to upload a php file into the torrent as an image and get code execution on the webserver.

We visit the pre existing torrent that is a kali linux iso and `edit this torrent`.
![[Pasted image 20210708014604.png]]

We upload our malicious php file there with a reverse shell and click submit while intercepting it on burp.
![[Pasted image 20210708014911.png]]

We need to change the `content-type` from `application/php` to `image/png`.
![[Pasted image 20210708015353.png]]

We would see a successfully uploaded page.
![[Pasted image 20210708015504.png]]

We could now click on the image to get a reverse shell on the box.
![[Pasted image 20210708015550.png]]

We get a shell on the box as `www-data`.
![[Pasted image 20210708015624.png]]

## Getting the user Flag
```python
www-data@popcorn:/var/www/torrent$ cd /home/george/
www-data@popcorn:/home/george$ ls -l
total 836
-rw-r--r-- 1 george george 848727 Mar 17  2017 torrenthoster.zip
-rw-r--r-- 1 george george     33 Jul  7 20:32 user.txt
www-data@popcorn:/home/george$ cat user.txt 
1149ded7215b4cf314a9864c8d535993
```

# Root Escalation
The box is pretty old and is running ubuntu `9.10` therefore it is vulnerable to dirty cow exploit.
![[Pasted image 20210708154945.png]]

We could get the exploit from : https://raw.githubusercontent.com/FireFart/dirtycow/master/dirty.c

We have to compile this exploit on the target machine and run it and on successfull exploitation it will create us a user named `firefart` with uid `0` with the password we want.

![[Pasted image 20210708155349.png]]

Now, We SSH into the box with as the `firefart` user and the password we set.

## Getting the root Flag
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/popcorn]
└──╼ # ssh firefart@10.10.10.6
firefart@10.10.10.6's password: 
Linux popcorn 2.6.31-14-generic-pae #48-Ubuntu SMP Fri Oct 16 15:22:42 UTC 2009 i686

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/

  System information as of Thu Jul  8 13:26:27 EEST 2021

  System load: 0.0               Memory usage: 11%   Processes:       118
  Usage of /:  7.9% of 14.80GB   Swap usage:   0%    Users logged in: 0

  Graph this data and manage this system at https://landscape.canonical.com/

Last login: Thu Jul  8 13:14:06 2021 from 10.10.14.18
firefart@popcorn:~# id
uid=0(firefart) gid=0(root) groups=0(root)
firefart@popcorn:~# ls -l
total 4
-rw------- 1 firefart root 33 2021-07-08 12:33 root.txt
firefart@popcorn:~# cat root.txt 
ad4d89a6aabf9918207003e01d1f347a
```