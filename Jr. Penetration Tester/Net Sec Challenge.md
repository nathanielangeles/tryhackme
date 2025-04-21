# TryHackMe Net Sec Challenge

## Overview

This writeup covers the TryHackMe Net Sec Challenge, which involves port scanning, banner grabbing, FTP enumeration, and retrieving hidden flags. The challenge is designed to test fundamental network enumeration and exploitation techniques using common tools.

## Tools Used

- `rustscan` (for fast port scanning)
- `nmap` (for service/version detection)
- `telnet` (for manual banner grabbing)
- `hydra` (for brute-forcing FTP credentials)

## Enumeration

Initial port scan using Rustscan:

```bash
rustscan -a 10.10.172.21
```

![answer1-2-3](https://github.com/user-attachments/assets/1b33c0c4-16d7-4e78-8022-d68252d5850a)

This revealed the following:

- Highest open port under 10,000: **8080**
- An open port above 10,000: **10021**
- Total TCP ports open: **6** (as per scan results)

## Banner Grabbing

### HTTP Header Flag
Using Telnet to check HTTP headers:

```bash
telnet 10.10.172.21 80
HEAD / HTTP/1.1
Host: telnet
```
![answer4](https://github.com/user-attachments/assets/bbfb8f02-93f4-44f3-9710-d4110e8d5df5)

**Flag found in HTTP header**: `THM{web_server_25352}`

### SSH Header Flag

```bash
telnet 10.10.172.21 22
```

![answer5](https://github.com/user-attachments/assets/5fdc2cd5-be3f-43a0-82ea-872ccec75e86)

**Flag found in SSH banner**: `THM{946219583339}`

## FTP Enumeration

The FTP server is running on port `10021`. To identify the service version:

```bash
sudo nmap -sS 10.10.172.21 -p10021 -T4 -A
```

![answer6](https://github.com/user-attachments/assets/25a434bd-bd1d-4e40-88bd-016c839ec50f)

**FTP Version**: `vsftpd 3.0.3`

## Brute Forcing FTP Login

We identified potential usernames via social engineering: `quinn` and `eddie`. To brute-force the password:

```bash
hydra -l quinn -P /usr/share/wordlists/rockyou.txt 10.10.172.21 ftp -s 10021
```

![method7](https://github.com/user-attachments/assets/8599f4f2-c7cc-46f9-8524-23a4277670c0)

Once credentials were obtained, we logged in:

```bash
ftp 10.10.172.21 -P 10021
```

![method7-2](https://github.com/user-attachments/assets/265bcb26-335a-482c-8d72-7bd2c0b35f5c)

Downloaded the flag file:

```bash
cat ftp_flag.txt
```

![method7-3](https://github.com/user-attachments/assets/754133e2-632b-407f-b412-ecdfb4107a25)

**Flag from FTP user files**: `THM{321452667098}`

## HTTP Challenge (Port 8080)

Visiting the following URL showed a web-based challenge:

```
http://10.10.172.21:8080
```
This challenge was discovered using:

```bash
sudo nmap -sN 10.10.172.21
```

![answer8](https://github.com/user-attachments/assets/375d89f2-21f5-49fe-a0f2-b126ecdb1f79)


Solving the puzzle revealed another flag.

**Flag from port 8080**: `THM{f7443f99}`

## Conclusion

This challenge demonstrated a variety of basic techniques in network security:

- High-speed port scanning with Rustscan
- Manual service enumeration
- FTP brute-forcing and file retrieval
- Identifying hidden data in server banners

It's an excellent exercise for building foundational network pentesting skills.

