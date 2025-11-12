# Room Name - Jerry

**Author:** Garrett Segura (@grehaus)

**Date:** 2025/11/11

**Difficulty:** Easy

**OS:**  Windows

**Summary**: An easy Windows machine showing the dangerous nature of default/common credentials paired with a metasploit exploit to gain NT AUTHORITY\SYSTEM access to the machine.

---

## Table of Contents
1. Report Summary
2. Scope and Rules
3. Recon
4. Enumeration
5. Exploitation
6. Initial Access
7. Mitigations and Recommendations

---

## 1. Report Summary

The /manager/html endpoint on the Apache Tomcat web server reveals credentials after failing the login attempt several times. A metasploit exploit can be used to target the webserver and get a shell as NT AUTHORITY\SYSTEM.

---

## 2. Scope and Rules

### Scope
**Target**: 10.129.10.201

### Rules
**Permitted Testing**: Recon, Exploitation, Post-Exploitation of the Jerry(10.129.10.201) machine. 

**Non-Permitted Testing**: Hack the Box infrastructure, other players, any security testing outside of the Hack the Box platform while connected to their network.

**Disclosure**: This write up contains a full walkthrough. However, flags, file contents, and credentials will not be included unless otherwise provided from the start. I highly encourage individuals to try on their own and use this guide as a last resort. Hack the Box takes their platform seriously and I will uphold their terms of service and guidelines, which can be found here https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines. 

---

## 3. Recon

`nmap -p- -vv -oN scans/nmap 10.129.10.201`

<img width="527" height="116" alt="image" src="https://github.com/user-attachments/assets/44f1e4ae-6f3c-4791-8224-ca5e8b9c63fe" />




Port 8080 is open running Apache Tomcat, continuously failing the login attempt at /manager/html shows the following which contains credentials - 

<img width="997" height="209" alt="image" src="https://github.com/user-attachments/assets/abcd0330-f6bb-4181-9850-e11a69357963" />


This endpoint can easily be found by running directory fuzzing programs or manual enumeration of the site.

---

## 4. Enumeration

Researching potential exploits for the Apache Tomcat server I stumbled upon this article - 
https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/tomcat/index.html
<img width="779" height="195" alt="image" src="https://github.com/user-attachments/assets/4a3b19cb-69c2-4fb8-b3e8-57e3ade8cae1" />

---

## 5. Exploitation

Using metasploit  with the `exploit/multi/http/tomcat_mgr_upload` module.
<img width="939" height="556" alt="image" src="https://github.com/user-attachments/assets/5324789d-d11d-4085-9b50-efb670b14025" />

---

## 6. Initial Accesss (Full Access as NT AUTHORITY\SYSTEM)

Running `whoami` on the machine shows I have access as the NT AUTHORITY\SYSTEM account.
<img width="585" height="819" alt="image" src="https://github.com/user-attachments/assets/f502e0c4-da31-4a42-bd96-b3657fd947ba" />

---

## 7. Mitigations and Recommendations
- Do not disclose sensitive information on public facing services.
- Use strong credentials when creating accounts, at minimum - one lowercase letter, one uppercase letter, one digit, one special character, and twelve characters in length.
- Keep software/programs up to date to mitigate vulnerabilities. 
