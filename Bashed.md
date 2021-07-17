# Bashed
>Author : Xyan1d3
>Date : 24th June 2021
>IP : 10.10.10.68
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Thu Jun 24 01:00:15 2021 as: nmap -sC -sV -v -oN nmap/bashed 10.10.10.68
Increasing send delay for 10.10.10.68 from 0 to 5 due to 40 out of 132 dropped probes since last increase.
Nmap scan report for bashed.htb (10.10.10.68)
Host is up (0.073s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 6AA5034A553DFA77C3B2C7B4C26CF870
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site
```

## Gobuster
```python
┌─[Magisk@Xyan1d3]─[10.10.14.15]─[~/htb/bashed]
└──╼ # gobuster dir -u http://10.10.10.68/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.out
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.68/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/24 01:04:49 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 311] [--> http://10.10.10.68/images/]
/uploads              (Status: 301) [Size: 312] [--> http://10.10.10.68/uploads/]
/php                  (Status: 301) [Size: 308] [--> http://10.10.10.68/php/]    
/css                  (Status: 301) [Size: 308] [--> http://10.10.10.68/css/]    
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.68/dev/]    
/js                   (Status: 301) [Size: 307] [--> http://10.10.10.68/js/]     
/fonts                (Status: 301) [Size: 310] [--> http://10.10.10.68/fonts/]
```

## Web Enumeration
We visit the site and find that the site is showing of something called a `phpbash.php`.
![[Pasted image 20210624010750.png]]

We try the to find it inside all of the directories and find out that we have it at `http://10.10.10.68/dev/phpbash.php`.
![[Pasted image 20210624011007.png]]

# Initial FootHold
We could convert this webshell like thingy to a proper reverse shell.
For, This we create an bash script and put a `bash -i` reverse shell and host it using the python3 `http.server` and on the webshell download it and execute it.
![[Pasted image 20210624011649.png]]

The `curl` command is not present on the box. So, We use `wget -O-` instead and pipe it over to bash to get a shell as `www-data`.
![[Pasted image 20210624011937.png]]
![[Pasted image 20210624011949.png]]

# User Escalation to Script Manager user
We do a simple `sudo -l` to find out that we have `nopasswd` on all command as `scriptmanager` user from `www-data`.
![[Pasted image 20210624012540.png]]

We do a `sudo -u scriptmanager /bin/bash`
![[Pasted image 20210624012604.png]]

We visit the `/home/arrexel` directory and get our `user.txt`.
![[Pasted image 20210624012946.png]]

# Root Escalation
We enumerate a little and find out that we have a `scripts` at `/` which contains a `test.py` and `test.txt`.
![[Pasted image 20210624014143.png]]

The `test.py` is writable by the `scriptmanager` user and it opens a file `text.txt` and writes into it and its a little wierd that the file has user and and group owned by root.
![[Pasted image 20210624014258.png]]

Therefore, The script is somehow is been executed as `root` user in order to have the recent timestamp and the user and group own.
We run `pspy` to find out that a `cron` is executing the is file.
![[Pasted image 20210624014103.png]]

We could modify this python file and can get a reverse shell as the `root` user.
![[Pasted image 20210624015240.png]]

We get the `root.txt` from the `/root` directory.
![[Pasted image 20210624015459.png]]