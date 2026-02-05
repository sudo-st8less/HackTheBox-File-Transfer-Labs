### &#x1F6A9; HTB Pentester Path <br>
### File Transfers - Linux File Transfers Lab <br>
<mark>hook it up with a &#x2B50; if this helps!</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>
<br>

---

IP:
10.129.37.170

---
### Question 1:
Download the file flag.txt from the web root using Python from the Pwnbox. Submit the contents of the file as your answer.

Quick wget:

	$ wget http://10.129.37.170/flag.txt -O /home/htb-ac-830862/flag.txt
	--2025-09-11 17:12:37--  http://10.129.37.170/flag.txt
	Connecting to 10.129.37.170:80... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 33 [text/plain]
	Saving to: ‘/home/htb-ac-830862/flag.txt’
	
	/home/htb-ac-830862/flag.txt  100%[===============================================>]      33  --.-KB/s    in 0s      
	
	2025-09-11 17:12:37 (1.55 MB/s) - ‘/home/htb-ac-830862/flag.txt’ saved [33/33]


&#x1F6A9; found **5d21cf3da9c0--edit--9e2559f3ea50** in downloaded file.

---

### Question 2:
SSH to 10.129.37.170 (ACADEMY-MISC-NIX04) with user "htb-student" and password "HTB_@cademy_stdnt!"

Upload the attached file named upload_nix.zip to the target using the method of your choice. Once uploaded, SSH to the box, extract the file, and run "hasher <extracted file>" from the command line. Submit the generated hash as your answer.


Downloaded zip to pwnbox:

	$ wget -O /home/htb-ac-830862/upload_nix.zip 
	
	https://academy.hackthebox.com/storage/modules/24/upload_nix.zip
	--2025-09-11 17:19:28--  https://academy.hackthebox.com/storage/modules/24/upload_nix.zip
	Resolving academy.hackthebox.com (academy.hackthebox.com)... 109.176.239.69, 109.176.239.70
	Connecting to academy.hackthebox.com (academy.hackthebox.com)|109.176.239.69|:443... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 194 [application/zip]
	Saving to: ‘/home/htb-ac-830862/upload_nix.zip’
	
	/home/htb-ac-830862 100%[===================>]     194  --.-KB/s    in 0s      
	
	2025-09-11 17:19:29 (3.89 MB/s) - ‘/home/htb-ac-830862/upload_nix.zip’ saved [194/194]

Next you need to SSH into the target:

	$ ssh htb-student@10.129.37.170
	
	The authenticity of host '10.129.37.170 (10.129.37.170)' can't be established.
	ED25519 key fingerprint is SHA256:z4rcb3qcf0IdRnoTBNEJ4i8TlDystDA4uOJFxVcb41E.
	This key is not known by any other names.
	Are you sure you want to continue connecting (yes/no/[fingerprint])? yes 
	Warning: Permanently added '10.129.37.170' (ED25519) to the list of known hosts.
	htb-student@10.129.37.170's password: 
	Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-47-generic x86_64)

Now let's setup a new dir, mv the zip there, and setup a python server on our pwn box:

	$ mkdir http && cd http
	$ mv /home/htb-ac-830862/upload_nix.zip /home/htb-ac-830862/http/upload_nix.zip


Simple py server running in http dir:

	$ python3 -m http.server 5555
	Serving HTTP on 0.0.0.0 port 5555 (http://0.0.0.0:5555/) ...


Now let's download the zip on the target:

	htb-student@nix04:~$ wget -O /home/htb-student/upload_nix.zip http://10.10.14.79:5555/upload_nix.zip
	--2025-09-11 22:49:31--  http://10.10.14.79:5555/upload_nix.zip
	Connecting to 10.10.14.79:5555... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 194 [application/zip]
	Saving to: ‘/home/htb-student/upload_nix.zip’
	
	/home/htb-student/upload_nix. 100%[===============================================>]     194  --.-KB/s    in 0s      
	
	2025-09-11 22:49:31 (21.8 MB/s) - ‘/home/htb-student/upload_nix.zip’ saved [194/194]
	
	htb-student@nix04:~$ ls
	upload_nix.zip



Since the unzip package isn't installed on the target and we don't have sudo privileges, I checked if python was installed, then proceeded to use the python unzip module:

	htb-student@nix04:~$ which python3
	/usr/bin/python3
	
	htb-student@nix04:~$ python3 -m zipfile -e upload_nix.zip .
	
	htb-student@nix04:~$ ls
	upload_nix.txt  upload_nix.zip
	
	htb-student@nix04:~$ hasher upload_nix.txt
	159cfe5c65054--edit--a359c8b0

&#x1F6A9; found **159cfe5c6505--edit--61cfa359c8b0**.
