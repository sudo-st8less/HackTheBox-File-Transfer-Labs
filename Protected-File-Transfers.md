### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Protected Transfers <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Protected File Transfers



Pentest loot is sensitive — NTDS.dit, password hashes, internal config dumps. Encrypt the data before transfer, even if you're using SSH/HTTPS/SFTP. **Never** exfil real PII / financial / trade-secret data unless the client explicitly asks for it during a DLP test — use dummy data with the same shape instead.

Windows AES file encryption — use [Invoke-AESEncryption.ps1](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1) (AES-256 CBC, SHA256 key derivation, IV prepended to ciphertext).

Import + encrypt:

```diff
+ PS> Import-Module .\Invoke-AESEncryption.ps1
+ PS> Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
```

Outputs `scan-results.txt.aes`. Encrypt strings inline:

```diff
+ PS> Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Text "Secret Text"
+ PS> Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Text "<base64 ciphertext>"
```

Decrypt:

```diff
+ PS> Invoke-AESEncryption -Mode Decrypt -Key "p4ssw0rd" -Path .\scan-results.txt.aes
```

Linux file encryption — `openssl enc` (preinstalled almost everywhere). Pick a cipher, set high iter count, enable PBKDF2:

```diff
+ $ openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```

Decrypt with `-d`:

```diff
+ $ openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```

| Flag | Purpose |
|---|---|
| `-aes256` | AES-256-CBC cipher |
| `-iter 100000` | PBKDF2 iterations (slow brute force) |
| `-pbkdf2` | use PBKDF2 KDF (vs deprecated MD5-based default) |
| `-d` | decrypt mode |

Rules:

- **Unique** strong password per engagement — prevents one cracked pass from compromising every client's loot.
- Pair encryption with a secure transport (HTTPS, SFTP, SSH) when possible. Defense in depth.
- Verify with hash before/after transfer.
