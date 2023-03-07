# \[Medium] Agile

<table><thead><tr><th>Name</th><th>Info</th><th data-hidden></th></tr></thead><tbody><tr><td>IP</td><td><mark style="color:blue;">10.129.171.197</mark></td><td></td></tr><tr><td>Domain</td><td><mark style="color:blue;">superpass.htb</mark></td><td></td></tr></tbody></table>



{% code title="nmap_report.txt" %}
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
