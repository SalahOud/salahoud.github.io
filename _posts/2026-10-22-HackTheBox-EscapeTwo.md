---
title: HackTheBox EscapeTwo
date: 2026-10-22 01:00:00 +0100
categories: [Writeup]
tags: [Active Directory, AD CS] 
render_with_liquid: false
media_subpath: /assets/imgs/EscapeTwo/
image:
  path: 0.png
---

## Initial Access
    
Ran nmap scan:

We start this box with credentials for the following account: rose / KxEPkKe6R8su
![](1.png){: width="2500" height="1300"}

Usernames enumeration :
![](2.png){: width="2500" height="1300"}

We spray the password we have on the new usernames with no hits :
![](3.png){: width="2500" height="1300"}

AS-REP Roasting gives two hits :
![](4.png){: width="2500" height="1300"}

No luck cracking the hashes with rockyou :
![](5.png){: width="2500" height="1300"}

Enumerating the smb shares :
![](6.png){: width="2500" height="1300"}

We find two interesting files accounts.xlsx and accounting_2024.xlsx : 
![](7.png){: width="2500" height="1300"}

We get an error when trying to unzip the files (bad zipfile offset) :

![](8.png){: width="2500" height="1300"}
![](9.png){: width="2500" height="1300"}

After fixing the offset we can access the files and we find more credentials :
![](10.png){: width="2500" height="1300"}
![](11.png){: width="2500" height="1300"}

One of the credentials work with MSSQL :
![](12.png){: width="2500" height="1300"}

We connect with mssqlclient and we find we have xp_cmdshell :
![](13.png){: width="2500" height="1300"}

We get a reverse shell :
![](14.png){: width="2500" height="1300"}
![](15.png){: width="2500" height="1300"}

After some looking around we find an interesting config file with some credentials :
![](16.png){: width="2500" height="1300"}
![](17.png){: width="2500" height="1300"}

Password Spraying :
![](18.png){: width="2500" height="1300"}

And we find the user flag here.

## Road to Root

We fire up bloodhound (I already ran rusthound earlier) :
![](19.png){: width="2500" height="1300"}

Bloodhound points out a clear path to full doamin takeover from the user ryan :
![](20.png){: width="2500" height="1300"}

First we assign full control (GenericAll) over the ca_svc account to our user ryan :
![](21.png){: width="2500" height="1300"}

Next we add a KeyCredential to ca_svc :
![](22.png){: width="2500" height="1300"}
![](23.png){: width="2500" height="1300"}

Now we can enumerate vulnerable templates, we find a template that is vulnerable to ESC4 :
![](24.png){: width="2500" height="1300"}
![](25.png){: width="2500" height="1300"}

Because ca_svc has full control over the template we can make it vulnerable to ESC1 :
![](26.png){: width="2500" height="1300"}

Now we can request a domain administrator certificate :
![](27.png){: width="2500" height="1300"}

And finally we can get the NTLM hash of the Administrator :
![](28.png){: width="2500" height="1300"}

And we get the root flag !
![](29.png){: width="2500" height="1300"}
