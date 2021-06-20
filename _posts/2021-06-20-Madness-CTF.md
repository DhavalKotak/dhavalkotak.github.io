---
published: false
---

Bonjour people. This is a writeup of a room called Madness on tryhackme. It involves : 

- Stegnography
- Web

Room : [Madness CTF](https://tryhackme.com/room/madness)

Are you ready to RUMBLLLEEEEEE!!!!! Sorry. Lets get started.

## Nmap Scanning

> nmap -A IPADDR

We get the following ports open :

> 22 SSH                                                                                                     
  80 HTTP

Lets take a look at the website.

## Web Stuff and Hexeditor

There is a photo on the top and there seems to be some problem with it. Lets look at the source to check it.

As we see in the above image there is a ```thm.jpg``` image that is broken. So lets first download the images with wget and try to see whats wrong.

Well that is wierd. The image header is of a PNG instead of a JPG. So lets fix that with ```hexeditor```.

Change 

> 89 50 47 4E   0D 0A 1A 0A   00 00 00 01

To

> FF D8 FF E0   00 10 4A 46   49 46 00 01

Now lets look at the image.

We got a hidden directory for the website, so lets check it. 

We need a secret code for this. After checking the source of this page we found out that the code is between 0-99. And we also find that we have to give the number with ```secret``` as a parameter in the URL. To do this I made a shell script to loop through 0-99 and collect the response by requesting the website with different numbers as parameter. Here is the script :

```
#!/bin/sh
i=0
while [ $i -le 99 ]
do 
	curl -L http://IPADDR/th1s_1s_h1dd3n?secret=$i >> result
   	i = `expr $i + 1`
done
```

After running this we get a file called result with all the responses from the website. After checking it we see our lucky number.

## Stegnography

This one took a while for me to figure out as i was trying to decryprt the text. But it had a different purpose. It was a passphrase for the ```thm.jpg``` image we got. Lets run stegseek and enter the passphrase.

After deciphering it with ROT13 the username we get is ```joker```. But now what? We don't have a password. This was the moment when the creater of the room was laughing at us. You can see that there is an image on the room page on tryhackme right? You have to steg that image to get the password

## SSH Login

We get the user flag right away after logging in

## Privilege Escalation

After running ```find /bin -perm -4000``` I saw a script called ```screen-4.5.0```. So i looked online and found an exploit right away that gives us root privileges. You can check it [here](https://www.exploit-db.com/exploits/41154). I made a script in ```/tmp``` and made it executable and after running it I had root.

And that was it. The challenge is completed. This box was really fun tbh. Hope you enjoyed the it as well. See you next time :)


