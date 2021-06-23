# Blue
>Author : Xyan1d3
>Date : 23rd June 2021
>IP : 10.10.10.40
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jun 23 16:38:04 2021 as: nmap -sC -sV -v -oN nmap/blue 10.10.10.40
Nmap scan report for blue.htb (10.10.10.40)
Host is up (0.076s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -19m44s, deviation: 34m37s, median: 14s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-06-23T12:09:29+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-06-23T11:09:30
|_  start_date: 2021-06-23T11:05:27
```

This is a Windows 7 machine as stated by the `smb-os-discovery` script from nmap.
OS Fingerprint : `Windows 7 Professional 7601 Service Pack 1`.

As, It is `Windows 7 build 7601` we should run the nmap `vuln` script on this box.
According to my knowledge it is likely vulnerable to `MS17-010` or `Eternal Blue`.

```sql
# Nmap 7.91 scan initiated Wed Jun 23 16:39:27 2021 as: nmap -sC -sV -oN nmap/blue-vuln --script vuln -v 10.10.10.40
Nmap scan report for blue.htb (10.10.10.40)
Host is up (0.079s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
```

And This Script converts my hopes into reality.

# Exploiting MS17-010
We will be using the `MS17-010` auto blue exploit to exploit this eternal blue exploit.
Source : https://github.com/3ndG4me/AutoBlue-MS17-010.git

We would have a directory structure looking like this :
```python
┌─[Magisk@Xyan1d3]─[10.10.14.15]─[~/htb/blue/AutoBlue-MS17-010]
└──╼ # ls -l
total 216
-rw-r--r-- 1 root root 26444 Jun 23 15:59 eternalblue_exploit10.py
-rw-r--r-- 1 root root 25741 Jun 23 15:59 eternalblue_exploit7.py
-rw-r--r-- 1 root root 24106 Jun 23 15:59 eternalblue_exploit8.py
-rw-r--r-- 1 root root  2801 Jun 23 15:59 eternal_checker.py
-rw-r--r-- 1 root root  1070 Jun 23 15:59 LICENSE
-rwxr-xr-x 1 root root  3853 Jun 23 15:59 listener_prep.sh
-rw-r--r-- 1 root root 25725 Jun 23 15:59 mysmb.py
-rw-r--r-- 1 root root 25388 Jun 23 16:50 mysmb.pyc
-rw-r--r-- 1 root root  5352 Jun 23 15:59 README.md
-rw-r--r-- 1 root root     8 Jun 23 15:59 requirements.txt
drwxr-xr-x 2 root root  4096 Jun 23 16:52 shellcode
-rw-r--r-- 1 root root 49249 Jun 23 15:59 zzz_exploit.py
```

## Generating Shellcode
We have to get inside the `shellcode` directory and generate the shellcode corresponding to our attackbox by running `shell_prep.sh`.
![[Pasted image 20210623174259.png]]

## Running The Exploit
We could now run the exploit which is the `eternalblue_exploit7.py` with some custom groom size : `17`.
![[Pasted image 20210623174844.png]]
**Setting The Groom Size to `17` is very essential for this exploitation.**

## User Flag
We can now get our `user.txt` by navigating `C:\Users\Haris\Desktop` directory.
![[Pasted image 20210623175328.png]]

## Root Flag
We can now get our `user.txt` by navigating `C:\Users\Administrator\Desktop` directory.
![[Pasted image 20210623175510.png]]