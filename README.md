# Road To OSCP
|Box|OS|Date|Difficulty|Learnings|
|--|--|--|--|--|
|Lame|Linux Ubuntu 8.04 Hardy|22/06/2021|Easy|Ancient SMB Version running and vulnerable to Bufferoverflow|
|Brainfuck|Linux Ubuntu 16.04 Xenial |22/06/2021|Insane| <ul><li>Wordpress Support Responsive Ticket Plugin Login Bypass</li><li>SMTP Email Enumeration with Evolution</li><li>One Time Pad & Vigenere Cipher Attack</li><li>RSA Attack by decoding the flag given p,q,e and c</li></ul>|
|Shocker|Linux Ubuntu 16.04 Xenial|22/06/2021|Easy|<ul><li>ShellShock on User-Agent on apache2 /cgi-bin/user.sh</li><li>sudo nopasswd on perl</li></ul>|
|Legacy|Windows XP SP3|23/06/2021|Easy|<ul><li>Exploiting `MS08-067`</li></ul>https://raw.githubusercontent.com/jivoi/pentest/master/exploit_win/ms08-067.py|
|Blue|Windows 7 build 7601 SP1 x64|23/06/2021|Easy|<ul><li>Exploiting `MS17-010`</li><li>Using Exploit AutoBlue from Github with 17 Groom Bytes</li></ul>https://github.com/3ndG4me/AutoBlue-MS17-010.git|
|Traverxec|Linux Debian Buster|23/06/2021|Easy|<ul><li>Nostromo 1.9.6 Exploitation</li><li>public_www directory in the user `~david`</li><li>Journalctl terminal shrink exploit</li></ul>|
|Bashed|Linux Ubuntu 16.04.2 Xenial|24/06/2021|Easy|<ul><li>PHPBash Exploitation</li><li>Sudo Nopasswd ALL</li><li>Cron python script Exploitation</li></ul>|
|Nibbles|Linux Ubuntu 16.04.3 Xenial|27/06/2021|Easy|<ul><li>Simple php upload in image upload(No Bypass)</li><li>Sudo Nopasswd on a bash script on the home dir of regular user</li></ul>|
|Devel|Windows 7 Enterprise x86 Build 7600|27/06/2021|Easy|<ul><li>IIS `www-data` hosted as anonymous FTP with write access</li><li>`MS11-046` Exploitation to `NT Authority/SYSTEM`</li></ul>|
|Optimum|Windows Server 2012 R2 x64|28/06/2021|Easy|<ul><li>Exploiting Rejetto HttpFileServer 2.3 URCE</li><li>Exploiting `MS16-098` to get `NT Authority/SYSTEM`</li></ul>|
|Bastard|Windows Server 2008 R2 Datacenter x64|28/06/2021|Medium|<ul><li>Exploiting Drupal `7.54` with Drupalgeddon2</li><li>Privilege Escalation with `MS10-059`(`Chimmichurri`)</li></ul>|
|Granny|Windows Server 2003 SP2 x86|29/06/2021|Easy|<ul><li>Exploiting IIS 6.0 WebDav BufferOverflow</li><li>Privilege Escalation by exploiting `MS09-012`</li></ul>|
|Silo|Windows Server 2012 R2 Standard Build 9600 x64|29/06/2021|Medium|<ul><li>Oracle Database exploitation with `odat`</li></ul>|
|Beep|CentOS 5.6|30/06/2021|Easy|<ul><li>LFI in PBX Elastix</li><li>SSH No exchange Algorithms(Old SSH Version)</li></ul>|
|Cronos|Linux Ubuntu 16.04.2 Xenial|30/06/2021|Medium|<ul><li>`dig`ing to find out domains</li><li>Cron Exploitation `root` exec `www-data` writable</li></ul>|
|Nineveh|Linux Ubuntu 16.04.2 Xenial|02/07/2021|Medium|<ul><li>Hydra Post-form brutforce on http & https</li><li>Stego on an image with `binwalk`</li><li>Port Knocking firewall 3 ports to SYN to open `22/tcp`</li><li>chkrootkit exploitation for root escalation</li></ul>|
|Sense|FreeBSD 8.3-RELEASE-p16|02/07/2021|Easy|<ul><li>Fuzzing for txt files on webserver to find out a file with creds to pfsense router</li><li>pfsense router version `2.1.3` vulnerable to RCE to root</li></ul>|
|Solidstate|Linux Debian 9 Strech|02/07/2021|Medium|<ul><li>Exploiting Apache JAMES with default creds and reseting all POP3 account creds</li><li>POP3 Enumeration</li><li>Escaping rbash SSH</li><li>Cron Exploitation</li></ul>|
|Node|Linux Ubuntu 16.04.3 Xenial|05/07/2021|Medium|<ul><li>Exploiting Node.js application due to exposing of creds on an api</li><li>Node.js scheduler app uses mongo to execute a command from a key</li><li>Root escalation with `ret2libc` or wildcard injection with `/roo*/roo*`,`/roo?/ro??.txt` & `/roo[a-z]/roo[a-z].txt`</li></ul>|
|Valentine|Linux Ubuntu 12.04 Precise|05/07/2021|Easy|<ul><li>Exploiting SSL Heartbleed attack</li><li>Attaching to tmux session socket</li></ul>|
|Poison|FreeBSD|05/07/2021|Medium|<ul><li>Discovering php file that lists file in `www-data` to leak pass to SSH</li><li>Using junk file to vnc into the box by tunneling</li></ul>|
|Tartarsauce|Linux Ubuntu 16.04.4 Xenial|06/07/2021|Medium|<ul><li>Gwolle-gb RFI exploitation</li><li>sudo user escalation tar</li><li>tar archive symlink injection</li></ul>|
|Irked|Linux Debian 8 Jessie|06/07/2021|Easy|<ul><li>Exploiting Unreal IRCd</li><li>Setuid binary systeming a file</li></ul>|
|Networked|Linux CentOS 7|07/07/2021|Easy|<ul><li>Image upload bypass with mime, `GIF89a` and `php.png`</li><li>php cron unsafe data shoved in system</li><li>Bash network script injection</li></ul>|
|Jarvis|Linux Debian 9 Strech|07/07/2021|Medium|<ul><li>SQLi with sqlmap os-shell after bypassing WAF with User-Agent check</li><li>bypassing bad character filtering and shoving to ping command</li><li>Setuid on systemctl</li></ul>|
|Mirai|Linux Debian 8 Jessie|07/07/2021|Easy|<ul><li>Using default creds of raspberry pi</li><li>Recovering deleted `root.txt` with `grep`</li></ul>|
|Popcorn|Linux Ubuntu 9.10 Karmic|08/07/2021|Medium|<ul><li>Php image upload bypass torrent hoster</li><li>Motd legal exploit or Dirty Cow</li></ul>|
|Haircut|Linux Ubuntu 16.04.2 Xenial|08/07/2021|Medium|<ul><li>Website shoves url to a curl request with poor filtering where backtick and $ works</li><li>Suid on Screen-4.5.0</li></ul>|
|Blocky|Linux Ubuntu 16.04.2 Xenial|09/07/2021|Easy|<ul><li>Discovering a jar file and decompiling it with `JD-GUI` and getting creds</li><li>sudo `ALL=(ALL)`</li></ul>|
|Frolic|Linux Ubuntu 16.04.4 Xenial|09/07/2021|Easy|<ul><li>Huge cipher enumeration and playsms exploitation with csv injection</li><li>ret2libc exploit as suid with ASLR disabled</li></ul>|
|Jerry|Windows Server 2012 R2|10/07/2021|Easy|<ul><li>Tomcat default creds manager war upload shell as `NT Authority/SYSTEM`</li></ul>|
|Bounty|Windows Server 2008 R2|10/07/2021|Easy|<ul><li>Bypassing file upload of by uploading `web.config` instead of asp/aspx</li><li>MS10-059 to `NT Authority/SYSTEM`</li></ul>|
|Conceal|Windows 10 Enterprise x64 1703|10/07/2021-11/07/2021|Hard|<ul><li>Enumerating SNMP</li><li>Enumerating 500/UDP IPSEC VPN to expose the open TCP ports</li><li>ALPC Exploit to privesc and Juicy Potato</li></ul>|
|Chatterbox|Windows 7 SP1 build 7601 x86|11/07/2021|Medium|<ul><li>AChat prewritten BufferOverflow Exploit</li><li>`icacls` to change the ownership of the `root.txt`</li></ul>|
|Bastion|Windows Server 2016 build 14393|11/07/2021|Easy|<ul><li>Mounting a `vhd` file on linux from smb</li><li>mRemoteng password decrypt / Print Nightmare</li></ul>|
|Curling|Linux Ubuntu 18.04 Bionic|12/07/2021|Easy|<ul><li>Web comment enumeration to login to joomla and template poisoning</li><li>`curl -K` on file exploitation</li></ul>|
|Access|Windows Server 2008 R2|12/07/2021|Easy|<ul><li>Microsoft Access Database file enumeration `mdb`</li><li>Outlook mailbox backup enumeration `mbox` and `pst`</li><li>Runas escalation with saved credentials</li></ul>|
|Sizzle|Windows Server 2016|14/07/2021|Insane|<ul><li>`scf` file upload to public desktop and responder hash</li><li>Powershell Constrained Language and Applocker bypass</li><li>Kerberoasting & DCSync</li></ul>|
|Resolute|Windows Server 2016|16/07/2021|Medium|<ul><li>RPC Enumeration from description</li><li>Hidden files in `C:\`</li><li>`DNS Admins` group Escalation</li></ul>|\
|Writeup|Debian Strech|16/07/2021|Easy|<ul><li>CMS made Simple SQLi</li>PATH Injection vulnerability<li></li></ul>|
|Mantis|Windows Server 2008|16/07/2021|Hard|<ul><li>`MS-SQL` Enumeration with `mssqlclient.py` and extracting credentials</li><li>Abusing `MS14-068` using impacket's `GoldenPac.py`</li></ul>|
|Mango|Linux Ubuntu 18.04.2 Bionic|17/07/2021|Medium|<ul><li>`No-SQL` regex injection to dump creds</li><li>Exploiting Setuid with `jjs`</li></ul>|
|Nest|Windows Server 2008 R2|17/07/2021|Easy|<ul><li>SMB Enumeration</li><li>`VB.Net` and `C#` code analyzing</li></ul>|
|Jeeves|Windows 10 Pro build 10586|18/07/2021|Medium|<ul><li>Jenkins Shell with No Login</li><li>Juicy Potato to `NT Authority/SYSTEM`</li></ul>|
|Devoops|Linux Ubuntu 16.04.4 Xenial|18/07/2021|Medium|<ul><li>XXE Injection</li><li>Git commit history Enumeration</li></ul>|
|Registry|Linux Ubuntu 18.04.3 Bionic|18/07/2021|Hard|<ul><li>Docker Registry Enumeration</li><li>Bolt CMS Enumeration</li><li>User to `www-data` descalation</li><li>sudo nopasswd on `restic` </li></ul>|
|Reel|Windows Active Directory|19/07/2021|Hard|<ul><li>RTF Dynamite</li><li>AppLocker Bypass</li><li>BloodHound Enumaeration</li><li>Using `Powerview.ps1` to abuse `WriteOwner`, `WriteDACL`, `GenericWrite` privileges/ACL's</li></ul>|
|Cascade|Windows Active Directory|20/07/2021|Medium|<ul><li>Anonymous LDAP Enumeration to find password</li><li>C# Binary Reverse Enggineering</li><li>`AD Recycle Bin` group Abuse</li></ul>|
|Rabbit|Windows Server 2008|20/07/2021|Insane|<ul><li>Complaint Management System SQL injection with `sqlmap`</li><li>Microsoft OWA 2010 otf exploit phishing</li><li>Exploiting `wamp` server running as `NT Authority/SYSTEM` by creating a php file in `www`</li></ul>|
|Toolbox|Debian 10 Buster|20/07/2021|Easy|<ul><li>SQLi to `sqlmap` and `--os-shell`</li><li>boot2docker abuse</li></ul>|
|Ariekei|Ubuntu 16.04.3 Xenial|21/07/2021|Insane|<ul><li>Docker Mesh Enumeration</li><li>Docker Group Abuse</li></ul>|
|Jail|CentOS 7.3|22/07/2021|Insane|<ul><li>Socket Reuse BufferOverflow</li><li>NFS with no root squashing</li><li>Custom Wordlist</li><li>Weak RSA key use `Restoring SSH Private Key from Public Key`</li></ul>|
