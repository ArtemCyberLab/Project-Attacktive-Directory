Project Objective

The objective of this project was to assess the security of an Active Directory domain infrastructure and achieve full domain compromise by exploiting common misconfigurations that are frequently abused in real-world enterprise attacks.

Project Description

During this project, I performed reconnaissance against the target system and identified it as a Windows Domain Controller exposing critical services such as Kerberos, LDAP, SMB, and RDP:

nmap -Pn -sC -sV -A -T4 -p- 10.65.184.209


After confirming the Domain Controller role, I enumerated valid domain users via unauthenticated Kerberos enumeration:

kerbrute userenum --dc 10.65.184.209 -d THM-AD users.txt


I discovered that one of the service accounts was vulnerable to AS-REP Roasting, allowing me to request a Kerberos authentication response without knowing the user’s password:

GetNPUsers.py THM-AD/svc-admin -no-pass -dc-ip 10.65.184.209


The extracted Kerberos hash was successfully cracked using a wordlist attack:

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt


Using the compromised domain credentials, I accessed available SMB shares and identified a backup share containing sensitive credential data:

smbclient //10.65.184.209/backup -U svc-admin


The discovered credentials were Base64-encoded and were successfully decoded:

echo YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw | base64 -d


With Domain Administrator privileges obtained, I performed a full dump of the Active Directory database (NTDS.dit), resulting in complete domain compromise:

secretsdump.py spookysec.local/backup:backup2517860@10.65.184.209

🧠 Conclusion

In this project, I successfully executed a full Active Directory attack chain, including Kerberos-based attacks, password cracking, SMB enumeration, and privilege escalation to Domain Administrator. This project demonstrates my practical understanding of Active Directory internals and common enterprise attack techniques used in real-world environments.
