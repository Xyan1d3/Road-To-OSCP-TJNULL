# Arctic
>Author : Xyan1d3
>Date : 29th June 2021
>IP : 10.10.10.11
>OS : Windows
>Difficulty : Easy

# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Tue Jun 29 12:17:39 2021 as: nmap -sC -sV -v -oN nmap/arctic 10.10.10.11
Nmap scan report for arctic.htb (10.10.10.11)
Host is up (0.076s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

