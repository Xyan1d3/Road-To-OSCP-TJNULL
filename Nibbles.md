# Nibbles
>Author : Xyan1d3
>Date : 27th June 2021
>IP : 10.10.10.75
>OS : Linux
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Fri Jun 25 20:34:25 2021 as: nmap -sC -sV -oN nmap/nibbles -v 10.10.10.75
Nmap scan report for nibbles.htb (10.10.10.75)
Host is up (0.083s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster
**Gobuster does not reveal anything useful here**

## Web Enumeration
Visiting the website we see that there is just an `hello world`.
![[Pasted image 20210625203953.png]]

We could now, check the source code of the page to see if something hidden there.
![[Pasted image 20210625204104.png]]

We see that there is an hidden web directory called `/nibbleblog/` which we can check for.

We visit the site and get a `Nibbles yum yum` page which looks like a CMS. Now, We could try to do a `gobuster` on this directory with php extensions.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.5]─[~/htb/nibbles]
└──╼ # gobuster dir -u http://10.10.10.75/nibbleblog/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 25 -x php
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.75/nibbleblog/
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2021/06/25 20:43:29 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 2986]
/sitemap.php          (Status: 200) [Size: 401] 
/content              (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/content/]
/themes               (Status: 301) [Size: 322] [--> http://10.10.10.75/nibbleblog/themes/] 
/feed.php             (Status: 200) [Size: 300]
/admin                (Status: 301) [Size: 321] [--> http://10.10.10.75/nibbleblog/admin/]  
/admin.php            (Status: 200) [Size: 1401]
/plugins              (Status: 301) [Size: 323] [--> http://10.10.10.75/nibbleblog/plugins/]
/install.php          (Status: 200) [Size: 78]
/update.php           (Status: 200) [Size: 1622]
/README               (Status: 200) [Size: 4628]
/languages            (Status: 301) [Size: 325] [--> http://10.10.10.75/nibbleblog/languages/]
```

We could now, check for `README` file as we could get CMS version from that file.
![[Pasted image 20210625205413.png]]

We see that the Nibbleblog CMS version `4.0.3` is running, Which we could search exploit. 
We have an Authenticated File Upload Vulnerability to be exact, which can be converted into Remote Command Exectution.
![[Pasted image 20210625211200.png]]

This part of the box really sucks because, we have to guess the credentials on our own.

|Username|Password|
|--|--|
|admin|nibbles|

![[Pasted image 20210627223014.png]]

# Initial Foothold as Nibbler
Now on look for any potential vulnerability for this CMS. And, We find one with which we can upload any arbitary file to the host and execute it. 
**Source : https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html**

According to the poc we could abuse the `my_image` plugin upload feature to upload a php revese shell and execute it on the webdirectory of the plugin.

According, to the poc we could to upload a malicious php script. By, Visiting the `my_image` plugin>configure section on the nibble blog page.

So, We visit there and upload a `laudanum` php reverse shell there and execute it on by visiting the `/content/private/plugins/my_image/` web directory.
![[Pasted image 20210627223648.png]]

We upload a file and get some php errors spitting out.
![[Pasted image 20210627223758.png]]
Now we start a listener and head over to `/content/private/plugins/my_image/` and execute the `image.php`.
**Any file uploaded will be renamed to `image.php`**

We recieve a shell as `nibbler`.
![[Pasted image 20210627224553.png]]

We can now get our `user.txt`.
![[Pasted image 20210627230109.png]]

# Root Escalation
We could do a simple `sudo -l` and find out that we have a sudo nopasswd on the file inside our home directory.
```python
nibbler@Nibbles:/home/nibbler$ sudo -l            
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

We could easily abuse it by making the file and putting in a simple bash reverse shell inside and adding the executable bit and executing it by sudo.
![[Pasted image 20210627232107.png]]

We recieve the shell as root and can get our root flag.
![[Pasted image 20210627232247.png]]

***