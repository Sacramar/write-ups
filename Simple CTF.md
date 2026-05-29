# TryHackMe - SimpleCTF

**Date:** 2026/01/06  
**Author:** Sacramar_  
**Platform:** TryHackMe  
**Room:** [SimpleCTF](https://tryhackme.com/room/easyctf)

---

## 🔍 Reconnaissance

First, I scanned the target with `nmap` to discover open ports and services.

![nmap scan result](https://github.com/user-attachments/assets/7e1c476f-5ffc-472b-8e74-5c1fe1288ca7)

The scan showed ports 21 (FTP), 80 (HTTP), and 2222 (SSH). I started with the web service.

Running `gobuster` and `nikto` against the site quickly revealed a directory called `/simple`. Browsing there showed it was a CMS Made Simple installation.

![web reconnaissance](https://github.com/user-attachments/assets/17e120bf-dcea-461f-9ee9-b4aae6658e26)

The `/simple` path led to the CMS's admin login page:

![admin login page](https://github.com/user-attachments/assets/25d93581-2ff5-46c9-820d-760fbb22ddb0)

---

## 💥 Exploitation

I recognized the CMS version as vulnerable to **CVE-2019-9053** – a time-based SQL injection in the News module. The original exploit was written for Python 2, so I adapted it for Python 3. The updated script is available in my [Pentest-tools repository](https://github.com/Sacramar/Pentest-tools/blob/main/CVE-2019%E2%80%939053%20(python3%20version)).

Running it dumped credentials from the database:

![CVE-2019-9053 output showing username, email, password, salt](https://github.com/user-attachments/assets/db6d339d-a51d-4d4c-a820-25f81877fbcf)

With the password cracked (using the salt), I tried to SSH. The server ran an old OpenSSH version that didn't support modern algorithms, so I had to specify legacy options:

```bash
ssh -p 2222 \
  -o KexAlgorithms=diffie-hellman-group14-sha1 \
  -o HostKeyAlgorithms=ssh-rsa \
  -o PubkeyAcceptedAlgorithms=ssh-rsa \
  mitch@<target-ip>
```

![successful SSH login](https://github.com/user-attachments/assets/572ef5b0-3558-4bee-b09e-caea1cff74dd)

Once logged in as `mitch`, I grabbed the user flag:

![user flag](https://github.com/user-attachments/assets/88c601ba-c0a9-4489-8393-bd0312f6c8c2)

---

## ⬆️ Privilege Escalation

I checked `sudo -l` and found that `mitch` could run `/usr/bin/vim` as root without a password.

```bash
sudo vim
```

Inside vim, I opened a shell with `:!bash` and immediately became root.

![root shell and root flag](https://github.com/user-attachments/assets/88c601ba-c0a9-4489-8393-bd0312f6c8c2)

Reading `/root/root.txt` completed the room.

---

## 🛡️ Vulnerability Assessment (CVSS v4.0)

| # | Vulnerability | CVSS v4.0 Vector | Score | Severity |
|---|---------------|------------------|:-----:|----------|
| 1 | **SQL Injection in CMS Made Simple (CVE-2019-9053)** | `CVSS:4.0/AV:N/AC:L/AT:P/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N` | **7.3** | 🟠 HIGH |
| 2 | **Deprecated SSH Algorithm (hmac-sha1)** | `CVSS:4.0/AV:N/AC:H/AT:P/PR:N/UI:N/VC:L/VI:L/VA:N/SC:N/SI:N/SA:N` | **5.3** | 🟡 MEDIUM |
| 3 | **Privilege Escalation via Sudo Vim (GTFOBins)** | `CVSS:4.0/AV:L/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:H/SC:H/SI:H/SA:H` | **8.8** | 🟠 HIGH |

---

## 🔧 Remediation

1. **Update CMS Made Simple** to the latest version. If an immediate update is not possible, deploy a Web Application Firewall (WAF) with SQL injection rules.
2. **Disable legacy SSH algorithms** in `/etc/ssh/sshd_config`. Allow only modern, secure algorithms (e.g., `hmac-sha2-256`, `hmac-sha2-512`).
3. **Restrict sudo permissions** – remove `mitch` from the sudo group or explicitly limit allowed commands. Avoid giving unrestricted `NOPASSWD` access to interpreters like `vim`.

---

## 💭 Final Thoughts

This was my very first CTF write-up, and I still remember the satisfaction of making that old Python 2 exploit work on Python 3. The room was straightforward, but it taught me a lot about dealing with outdated services and the importance of checking even basic misconfigurations like `sudo -l`.

> *“Everyone starts somewhere. This was my somewhere.”*
