# Paper
**Author:** Garrett Segura (@grehaus)

**Date:** 11/29/2025

**Difficulty:** Easy

**OS:**  Linux

**Summary**: An outdated WordPress version allows an unauthorized parties to leak information from secret content, exposing a chatbot for employees. The chat bot is misconfigured allowing files to be read outside the sales directory, which reveals user credentials. Gaining access to the machine and running automated enumeration identifies the polkit version is out of date and can be exploited with public scripts.

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
A custom X-Backend-Server reveals the server handling requests. After visiting the new found site, the bottom of the page shows that the WordPress CMS is being used for the site. The version is outdated and vulnerable to CVE-2019-17671, which upon using reveals a chatbot for employees. The chatbot is not secure and can be exploited to retrieve credentials from hidden files. Once an unauthorized user gains access from the leaked credentials, the outdated polkit service can be exploited to create a root level user and compromise the system.

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
<img width="336" height="79" alt="image" src="https://github.com/user-attachments/assets/56b5ccfd-594c-4464-9e74-d8b960392158" />

Initial scan shows ssh and http/https services running on the machine. 

----

<img width="522" height="187" alt="image" src="https://github.com/user-attachments/assets/203b704d-ac78-464c-b718-3b85864a5ebc" />

The "office.paper" server is shown in the X-Backend-Server header, this is added to the hosts file.

---

<img width="441" height="23" alt="image" src="https://github.com/user-attachments/assets/7280ede3-28b7-4023-87e3-e786cadde528" />

Now navigating to `http://office.paper` we can see at the bottom of the page the site is using WordPress.

---
## 4. Enumeration

<img width="957" height="75" alt="image" src="https://github.com/user-attachments/assets/a9634be7-0a5a-4774-b3eb-cd971fdd5a36" />

Using wpscan shows an outdated version, vulnerable to `CVE-2019-17671`
 - https://www.exploit-db.com/exploits/47690
---

<img width="1630" height="205" alt="image" src="https://github.com/user-attachments/assets/5a255b63-82fd-4077-9aca-9c87abbd5433" />

Using the CVE reveals an employee chat system.

---

<img width="818" height="219" alt="image" src="https://github.com/user-attachments/assets/faa6027b-59ba-4984-b7f1-6a29bd70a34d" />

After adding `chat.office.paper` to the hosts file, navigating to the link, creating a user, and looking through the message history, I stumbled upon a bot that can interact with commands.

---

<img width="402" height="296" alt="image" src="https://github.com/user-attachments/assets/d04fed36-f9c6-4408-b46f-11182864eef4" />

Testing the bot, we can see that it is indeed performing a long listing on the `sales` directory. However, the bot is not as secure as it mentions to be. We can escape the `sales` directory by a series of `../` . The bot also mentions reading files, which after digging around I found an environment file in the `hubot` directory.


We know there is a user `dwight`, this is the perfect time to try using known credentials with the previously found ssh service.

---

## 5. Initial Accesss

<img width="484" height="59" alt="image" src="https://github.com/user-attachments/assets/3e3def27-e98c-4e76-af17-463dadfbdd93" />

The found credentials lands us with shell access as `dwight`. 

---

<img width="673" height="149" alt="image" src="https://github.com/user-attachments/assets/00ce5d4c-9c1c-4141-9e4d-52c9f952f7f4" />

Setting up a Python3 server to retrieve the `linpeas.sh` file to run automatic enumeration.

---

<img width="1320" height="214" alt="image" src="https://github.com/user-attachments/assets/7c61b190-dfef-43ed-86e0-bb77377000be" />

There are a few interesting findings from linpeas, however, CVE-2021-3560 stands out. 

---
## 6. Privilege Escalation
CVE-2021-3560 can be used to effectively create our own root user.
- https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation/blob/main/poc.sh

---

<img width="765" height="630" alt="image" src="https://github.com/user-attachments/assets/85571d17-443c-489d-bf9b-5966743e8ea1" />

This is a time based attack against the dbus messaging, so it may take a few runs to properly execute. 

---

## 7. Mitigations and Recommendations

- Keep CMS patched and up to date.
- Implement strong authorization policies when using interactive bots.
- Keep system packages patched and up to date.
---

