---
published: true
---

Hello BURRRRPPPPP everyone!! This is a writeup of the Pickle Rick CTF. 

![]({{site.baseurl}}/images/pickle rick/room.jpeg)

Room : [Pickle Rick](https://tryhackme.com/room/picklerick)


This CTF is based on the Rick and Morty show which gives a little excitement. So lets DO IT! JUST DO IT!!

## Nmap Scanning 

> nmap -A IPADDR

We get the following ports open :

> 22 SSH                                                                                                     
  80 HTTP
  
## Nikto 

> nikto -h IPADDR

After runnig nikto we get a ```login.php``` page.

## Gobuster

> gobuster dir -u http://IPADDR -w /path/to/wordlist.txt

After running gobuster we get a ```robots.txt``` file and the usual ```index.html``` and a ```server-status``` returning 403. On checking the robots.txt file we get Wubbalubbadubdub. This might be a password but we still don't know. Now lets check the website

![]({{site.baseurl}}/images/pickle rick/website.png)

There isn't much going on here so lets move onto the source code.

![]({{site.baseurl}}/images/pickle rick/source.png)

And we have some sort of a username. Now lets try to login using the credential we found into the login.php page.

![]({{site.baseurl}}/images/pickle rick/login.png)

![]({{site.baseurl}}/images/pickle rick/shell.png)

And we have a shell(sort of). So lets try to ls.

![]({{site.baseurl}}/images/pickle rick/1st.png)

We get the first ingredient in the current directory. When we check the clue.txt file it say to check the file system for other ingredients. So it may be possible to perform [Directory Traversal](https://en.wikipedia.org/wiki/Directory_traversal_attack). So lets try to run the following command to move upto the root directory and list its content

> cd / ; ls -la 

![]({{site.baseurl}}/images/pickle rick/root_directory.png)

So it worked. Now lets check out the ```/home``` directory to look for user's directory.

> cd /home ; ls -la

![]({{site.baseurl}}/images/pickle rick/rick.png)

We see a user ```rick``` . So now lets move in his directory and check it.

> cd /home/rick ; ls -la

![]({{site.baseurl}}/images/pickle rick/2nd.png)

And we get the second ingredient after running 

> cd /home/rick ; less "second ingredients"

## Privelege Escalation

Now lets run ```sudo -l``` to check what we can run to get root.

![]({{site.baseurl}}/images/pickle rick/sudo.png)

WHOA! NOPASSWD : ALL ???? Seriously?? Ok then. We can check the ```/root``` directory without any password so why not.

> sudo ls -la /root

![]({{site.baseurl}}/images/pickle rick/3rd.png)

And we can read it as well. So challenge completed BURRRRPPPPP I guess. Pretty good room BURRRRPPPPP and also an easy one but was informative about directory traversal and command injection.
