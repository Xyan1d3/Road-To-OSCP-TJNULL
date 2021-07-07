# Mirai
>Author : Xyan1d3
>Date : 7th July 2021
>IP : 10.10.10.48
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jul  7 11:35:06 2021 as: nmap -sC -sV -v -oN nmap/mirai 10.10.10.48
Nmap scan report for mirai.htb (10.10.10.48)
Host is up (0.079s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http    lighttpd 1.4.35
|_http-favicon: Unknown favicon MD5: 2CE9555B2C4DF3AA682D512D2FA7C5E2
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: lighttpd/1.4.35
|_http-title: Website Blocked
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Nmap all ports no scripts
```sql
# Nmap 7.91 scan initiated Wed Jul  7 11:36:01 2021 as: nmap -p- -v -oN nmap/mirai-allports 10.10.10.48
Nmap scan report for mirai.htb (10.10.10.48)
Host is up (0.075s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
80/tcp    open  http
1513/tcp  open  fujitsu-dtc
32400/tcp open  plex
32469/tcp open  unknown
```

## Nmap all open ports with scripts
```sql
# Nmap 7.91 scan initiated Wed Jul  7 11:45:13 2021 as: nmap -p22,53,80,1513,32400,32469 -v -A -oN nmap/mirai-deepscan 10.10.10.48
Nmap scan report for mirai.htb (10.10.10.48)
Host is up (0.075s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp    open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp    open  http    lighttpd 1.4.35
|_http-favicon: Unknown favicon MD5: 2CE9555B2C4DF3AA682D512D2FA7C5E2
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: lighttpd/1.4.35
|_http-title: Website Blocked
1513/tcp  open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
32400/tcp open  http    Plex Media Server httpd
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-cors: HEAD GET POST PUT DELETE OPTIONS
|_http-favicon: Plex
|_http-title: Unauthorized
32469/tcp open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   73.00 ms 10.10.14.1
2   74.64 ms mirai.htb (10.10.10.48)

```

## Gobuster on `http://10.10.10.48/admin/`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/mirai]
└──╼ # gobuster dir -u http://10.10.10.48 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.48
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/07 12:04:19 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 0] [--> http://10.10.10.48/admin/]
/versions             (Status: 200) [Size: 13]
```

# Initial FootHold as pi
The box is running Pi-Hole `3.1.4` on `http://10.10.10.48/admin`.
![[Pasted image 20210707213231.png]]

As for most cases the pihole is to be installed on a raspberry pi.
Which has a default credential of `pi` and `raspberry`.

We try those credentials and get logged in to the box.
![[Pasted image 20210707213806.png]]

## Getting user Flag
```python
pi@raspberrypi:~ $ cd Desktop/
pi@raspberrypi:~/Desktop $ ls -l
total 8
drwxr-xr-x 4 pi pi 4096 Aug 13  2017 Plex
-rw-r--r-- 1 pi pi   32 Aug 13  2017 user.txt
pi@raspberrypi:~/Desktop $ cat user.txt; echo
ff837707441b257a20e32199d7c8838d
```

# Root Escalation
We do a `sudo -l` and find out that we have a sudo `nopasswd` on all commands.
```python
pi@raspberrypi:~/Desktop $ sudo -l
Matching Defaults entries for pi on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User pi may run the following commands on localhost:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
pi@raspberrypi:~/Desktop $ sudo -i

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

root@raspberrypi:~# id
uid=0(root) gid=0(root) groups=0(root)
```

## Getting the root Flag
```python
root@raspberrypi:~# ls
root.txt
root@raspberrypi:~# cat root.txt 
I lost my original root.txt! I think I may have a backup on my USB stick...
```
We could do a `df -h` to check for any usbdrives.
![[Pasted image 20210707222456.png]]

We could use a grep command to recover the file.
![[Pasted image 20210707221808.png]]
![[Pasted image 20210707221825.png]]

```python
root@raspberrypi:/media/usbstick# grep -a -B 25 -A 100 'root.txt' /dev/sdb

*** SNIP ***

(["       1YS1Y
               <Byc[B)>r &<yZ.Gum^>
                                   1Y
|}*,.+-3d3e483143ff12ec505d026fa13e020b
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?

-James
```