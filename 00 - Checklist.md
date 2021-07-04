# Checklist
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
|Valentine|Linux Ubuntu 12.04 Precise |05/07/2021|Easy|<ul><li>Exploiting SSL Heartbleed attack</li><li>Attaching to tmux session socket</li></ul>|