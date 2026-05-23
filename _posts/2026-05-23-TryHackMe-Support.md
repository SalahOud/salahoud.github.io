---
title: TryHackMe Support
date: 2026-05-23 01:00:00 +0100
categories: [Writeup]
tags: [LFI, RCE] 
render_with_liquid: false
media_subpath: /assets/imgs/support/
image:
  path: 0.png
---

## Reconnaissance
    
Ran nmap scan:
![](1.png){: width="2500" height="1300"}

We have a login page and a user : help@support.com 
![](2.png){: width="2500" height="1300"}

## First Part
After some fuzzing and looking around I decided to try bruteforcing the login and it worked we got the password for that user:
![](3.png){: width="2500" height="1300"}

Dir discovery :
![](3-1.png){: width="2500" height="1300"}

I checked out all those directories then I looked at the cookies, we have a isITUser cookie with an md5 hash
![](4.png){: width="2500" height="1300"}

Decrypting the hash:
![](4-2.png){: width="2500" height="1300"}

So we change it to true and we get an API admin panel:
![](5.png){: width="2500" height="1300"}

![](6.png){: width="2500" height="1300"}

![](7.png){: width="2500" height="1300"}

I tried messing with this api endpoint with POST, PUT, PATCH ...  requests but nothing seemed to work, so I went back the skin?= function looking for some LFI, eventully I remembred the files in the /includes path had .php extensions (like red.php), yet when we change the theme the url show skin=red, so i tried some comman files without the .php and finally noticed a change in the page with ../config : 
![](8.png){: width="2500" height="1300"}

We look at the source code and we get a password:
![](9.png){: width="2500" height="1300"}

I tried to login with the specialadmin@support.com account but it failed.
I was stuck here for hours, there were no writeups yet, so I could not look for a hint, I tried a lot of stuff, but the things is the password we got in the config.php was wrong ?? the password that worked is support110 not support@110. This was weird.

We login as admin and we the first flag
![](10.png){: width="2500" height="1300"}

## Second Part

First thing I noticed was the Date drop-down button that appeared after we login as admin, as the theme one was the path to the first answer I had to investigate this. So I went back to the php files looking for clues and founf this in  footer.php :
![](10-2.png){: width="2500" height="1300"}

The code uses shell_exec() on the sys POST parameter, only checking if the input string starts with date. This was our way to RCE.

We test it :

![](11.png){: width="2500" height="1300"}
![](12.png){: width="2500" height="1300"}

It works!

Now let's get a reverse shell :
![](13.png){: width="2500" height="1300"}

And we get the second flag:
![](14.png){: width="2500" height="1300"}






