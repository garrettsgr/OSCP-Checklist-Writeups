# Paper
**Author:** Garrett Segura
**Date:** 11/29/2025
**Difficulty:** Easy
**OS:**  Linux
**Summary**: An outdated WordPress version allows an unauthorized parties to leak information from secret content, exposing a chatbot for employees. The chat bot is vulnerable to reading files outside the set location, which reveals user credentials. Gaining access to the machine and running automated scripts locates the polkit version is out of date and can be exploited with public scripts.

---

## Table of Contents
1. Report Summary
2. Scope and Rules
3. Recon
4. Enumeration
5. Initial Access
6. Privilege Escalation
7. Mitigations and Recommendations

---

## 1. Report Summary
A custom X-Backend-Server reveals the server handling requests. After visiting the new found site, the bottom of the page shows that the WordPress CMS is being used for the site. The version is outdated and vulnerable to `CVE-2019-17671`, which upon using reveals a chatbot for employees. The chatbot is not secure and can be exploited to retrieve credentials from hidden files. Once an unauthorized user gains access from the leaked credentials, the outdated polkit service can be exploited to create a root level user and compromise the system.

---

## 2. Scope and Rules

#### Scope
**Target**: 10.129.136.31

#### Rules
**Permitted Testing**: Recon, Exploitation, Post-Exploitation of the Paper (10.129.136.31) machine. 
**Non-Permitted Testing**: Hack the Box infrastructure, other players, any security testing outside of the Hack the Box platform while connected to their network.
**Disclosure**: This write up contains a full walkthrough. However, flags, file contents, and credentials will not be included unless otherwise provided from the start. I highly encourage individuals to try on their own and use this guide as a last resort. Hack the Box takes their platform seriously and I will uphold their terms of service and guidelines, which can be found here https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines. 

---

## 3. Recon
![[Pasted image 20251129124829.png]]
Initial scan shows ssh and http/https services running on the machine. 


![[Pasted image 20251129124942.png]]
The "office.paper" server is shown in the X-Backend-Server header, this is added to the hosts file.


![[Pasted image 20251129125249.png]]
Now navigating to `http://office.paper` we can see at the bottom of the page the site is using WordPress.

---
## 4. Enumeration

![[Pasted image 20251129125949.png]]
Using wpscan shows an outdated version, vulnerable to `CVE-2019-17671`
 - https://www.exploit-db.com/exploits/47690


![[Pasted image 20251129130411.png]]
Using the CVE reveals an employee chat system.


![[Pasted image 20251129130839.png]]
After adding `chat.office.paper` to the hosts file, navigating to the link, creating a user, and looking through the message history, I stumbled upon a bot that can interact with commands.


![[Pasted image 20251129131042.png]]
Testing the bot, we can see that it is indeed performing a long listing on the `sales` directory. However, the bot is not as secure as it mentions to be. We can escape the `sales` directory by a series of `../` . The bot also mentions reading files, which after poking around I found an environment file in the `hubot` directory.


![[Pasted image 20251129131557.png]]
We know there is a user `dwight`, this is the perfect time to try using known credentials with the previously found ssh service.

---

## 5. Initial Accesss

![[Pasted image 20251129131829.png]]
The found credentials lands us with shell access as `dwight`. 


![[Pasted image 20251129132139.png]]
Setting up a Python3 server to retrieve the `linpeas.sh` file to run automatic enumeration.


![[Pasted image 20251129133228.png]]
There are a few interesting findings from linpeas, however, CVE-2021-3560 stands out. 

---
## 6. Privilege Escalation
CVE-2021-3560 can be used to effectively create our own root user.
- https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation/blob/main/poc.sh



![[Pasted image 20251129134019.png]]
This is a time based attack against the dbus messaging, so it may take a few runs to properly execute. 

---

## 7. Mitigations and Recommendations

- Keep CMS patched and up to date.
- Implement strong authorization policies when using interactive bots.
- Keep system packages patched and up to date.
---

