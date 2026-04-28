### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Intro <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### File Transfer Intro



File transfers are core to most engagements — moving tools onto targets, exfiltrating loot back. The path is rarely clean: AV/EDR, app whitelisting, web filtering, egress firewall rules, and IDS/IPS all interfere.

Common scenario: RCE on IIS via unrestricted file upload → web shell → reverse shell → enumerate priv esc. PowerShell blocked by app control policy → manual enum → `SeImpersonatePrivilege` found → need to drop `PrintSpoofer`. Certutil / FTP / GitHub all blocked by web content filter and egress rules. SMB (TCP/445) is allowed outbound → spin up `impacket-smbserver` → drop binary → escalate to admin.

Takeaways:

- Know multiple methods per OS (Windows + Linux), per protocol (HTTP/S, SMB, FTP, SSH, raw TCP).
- Match the method to the environment — what's blocked, what's monitored, what's allowed by default.
- Many module exercises and HTB boxes will require file transfer — practice every technique.
