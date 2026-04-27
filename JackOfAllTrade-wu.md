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

