# Friendzone
>Author : Xyan1d3
>Date : 6th July 2021
>IP : 10.10.10.123
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jul  6 17:01:01 2021 as: nmap -sC -sV -v -oN nmap/friendzone 10.10.10.123
Nmap scan report for 10.10.10.123
Host is up (0.060s latency).
Not shown: 993 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
|_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    Apache httpd 2.4.29
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 404 Not Found
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Issuer: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-10-05T21:02:30
| Not valid after:  2018-11-04T21:02:30
| MD5:   c144 1868 5e8b 468d fc7d 888b 1123 781c
|_SHA-1: 88d2 e8ee 1c2c dbd3 ea55 2e5e cdd4 e94c 4c8b 9233
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## DNS Enumeration
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/friendzone]
└──╼ # dig axfr @10.10.10.123 friendzone.red

; <<>> DiG 9.16.15-Debian <<>> axfr @10.10.10.123 friendzone.red
; (1 server found)
;; global options: +cmd
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzone.red.         604800  IN      AAAA    ::1
friendzone.red.         604800  IN      NS      localhost.
friendzone.red.         604800  IN      A       127.0.0.1
administrator1.friendzone.red. 604800 IN A      127.0.0.1
hr.friendzone.red.      604800  IN      A       127.0.0.1
uploads.friendzone.red. 604800  IN      A       127.0.0.1
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 64 msec
;; SERVER: 10.10.10.123#53(10.10.10.123)
;; WHEN: Tue Jul 06 18:33:00 IST 2021
;; XFR size: 8 records (messages 1, bytes 289)
```
- No anonymous login on FTP.
- SMB null session attack lists a bunch of shares.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/friendzone]
└──╼ # smbmap -H 10.10.10.123
[+] Guest session       IP: 10.10.10.123:445    Name: 10.10.10.123
        Disk                          Permissions     Comment
        ----                          -----------     -------
        print$                        NO ACCESS       Printer Drivers
        Files                         NO ACCESS       FriendZone Samba Server Files /etc/Files
        general                       READ ONLY       FriendZone Samba Server Files
        Development                   READ, WRITE     FriendZone Samba Server Files
        IPC$                          NO ACCESS       IPC Service (FriendZone server (Samba, Ubuntu))

```

We could check the `general` share first and see what is inside that share.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/friendzone]
└──╼ # smbclient //10.10.10.123/general
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Jan 17 01:40:51 2019
  ..                                  D        0  Thu Jan 24 03:21:02 2019
  creds.txt                           N       57  Wed Oct 10 05:22:42 2018

                9221460 blocks of size 1024. 6460340 blocks available
smb: \> get creds.txt
getting file \creds.txt of size 57 as creds.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \> exit
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/friendzone]
└──╼ # cat creds.txt 
creds for the admin THING:

admin:WORKWORKHhallelujah@#
```

We have a possible credential pair.

|Username|Password|
|--|--|
|admin|WORKWORKHhallelujah@#|

The `Development` share is empty.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/friendzone/smb/development]
└──╼ # smbclient //10.10.10.123/development
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul  6 17:07:10 2021
  ..                                  D        0  Thu Jan 24 03:21:02 2019

                9221460 blocks of size 1024. 6460336 blocks available
```

## Web Enumeration
We visit the `http://10.10.10.123` on browser and find out that we have a page `Have you been friendzoned`.
![[Pasted image 20210706180244.png]]

We check the `robots.txt` and find out that it trollls us by saying `seriously ?!`.

Now, We take a look at the `https://10.10.10.123` and find out that we have self signed certificate which leaks a hostname.
![[Pasted image 20210706180602.png]]

The site says you are `Ready to escape from friendzone`.
![[Pasted image 20210706180819.png]]

We view the `source` of the page and find out that there is a hidden directory.
![[Pasted image 20210706181523.png]]

We could now check the `https://administrator1.friendzone.red` and find out that we have a login page which we could check the credentials we got from the smb share.
![[Pasted image 20210706184242.png]]

