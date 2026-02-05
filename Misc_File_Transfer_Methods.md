### &#x1F6A9; HTB Pentester Path <br>
### File Transfers - Misc File Transfer Methods Lab <br>
<mark>hook it up with a &#x2B50; if this helps!</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>
<br>

---

IP:
10.129.199.72

---
### Question 1:
Use xfreerdp or rdesktop to connect to the target machine via RDP (Username: htb-student | Password:HTB_@cademy_stdnt!) and mount a Linux directory to practice file transfer operations (upload and download) with your attack host. Type "DONE" when finished.


	$ xfreerdp /u:htb-student /p:HTB_@cademy_stdnt! /v:10.129.199.72 /d:HTB /drive:linux,/home/htb-ac-830862/Desktop/mayoasaservice


Went to Network tab in file explorer, dragged file to desktop.
