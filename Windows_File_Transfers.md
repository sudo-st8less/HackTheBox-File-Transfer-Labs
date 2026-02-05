### &#x1F6A9; HTB Pentester Path <br>
### File Transfers - Windows File Transfers Lab <br>
<mark>hook it up with a &#x2B50; if this helps!</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>
<br>

---

Target IP:
10.129.21.58

---
### Question 1:
Download the file flag.txt from the web root using wget from the Pwnbox. Submit the contents of the file as your answer.

	$ wget http://10.129.21.58/flag.txt


&#x1F6A9; found **b1a4ca918282--edit--1944a3b**.

---
### Question 2:
Upload the attached file named upload_win.zip to the target using the method of your choice. Once uploaded, unzip the archive, and run "hasher upload_win.txt" from the command line. Submit the generated hash as your answer.


First, I downloaded the zip file from htb to my pwnbox cwd, then base 64 encoded it:

	$ base64 -w 0 upload_win.zip
	UEsDBAoAAAAAAFmEKVFHXocmIAAAACAAAAAOAAAAdXBsb2FkX3dpbi50eHRlNGZlZWM0NjZkNWRlNzAxMDg5YjVjYzFiZjZkNTkyYVBLAQI/AAoAAAAAAFmEKVFHXocmIAAAACAAAAAOACQAAAAAAAAAIAAAAAAAAAB1cGxvYWRfd2luLnR4dAoAIAAAAAAAAQAYAHjm8KnohtYBzETj5fqG1gEXkIab6IbWAVBLBQYAAAAAAQABAGAAAABMAAAAAAA=



Then make a txt file of the hash in the home dir of the windows box, called zip64.txt

Now recreate the .zip using the zip64 text file on the windows user's home dir with PS.
First we make a variable that reads the b64 string:

	PS C:\Users\htb-student> $b64 = Get-Content -Path "C:\Users\htb-student\zip64.txt" -Raw

Now we convert it back to bytes:

	PS C:\Users\htb-student> $bytes = [Convert]::FromBase64String($b64)


Now we can write it to a file:

	PS C:\Users\htb-student> [IO.File]::WriteAllBytes("C:\Users\htb-student\upload_win.zip", $bytes)

And finally run hasher on the text file inside the zip, like the directions tell us to:

	hasher upload_win.txt
