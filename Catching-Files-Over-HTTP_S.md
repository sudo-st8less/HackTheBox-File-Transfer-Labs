### CPTS / HTB Penetration Tester Path <br>
### File Transfers - Catching Files over HTTP/S <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Catching Files over HTTP/S



HTTP/S is the universal transfer fallback — almost always allowed outbound, often encrypted in transit. Don't use Apache as an upload sink — its PHP module will execute anything ending in `.php`, which is how upload servers turn into web shells. Nginx's PHP support is opt-in, making it a safer default for catching files.

Setup directory + ownership:

```diff
+ $ sudo mkdir -p /var/www/uploads/SecretUploadDirectory
+ $ sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

Create `/etc/nginx/sites-available/upload.conf` — listen on 9001, enable `dav_methods PUT`:

```nginx
server {
    listen 9001;

    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

Enable the site, restart nginx:

```diff
+ $ sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
+ $ sudo systemctl restart nginx.service
```

If port 80 is already taken (Pwnbox runs `websockify` there), check error log + remove default config:

```diff
+ $ tail -2 /var/log/nginx/error.log
+ $ ss -lnpt | grep 80
+ $ sudo rm /etc/nginx/sites-enabled/default
```

Test with curl `-T` (PUT):

```diff
+ $ curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt
+ $ sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt
```

| Setting | Why |
|---|---|
| Nginx (not Apache) | no PHP-by-default → no accidental web shell |
| `dav_methods PUT` | enables WebDAV PUT only (no LIST / DELETE) |
| Hidden dir name | obscures the upload path from directory enum |
| No `autoindex on` | Nginx doesn't list dir contents by default |

Make sure to verify the upload dir is **not browsable** — sensitive files in there shouldn't be discoverable via simple URL guessing.
