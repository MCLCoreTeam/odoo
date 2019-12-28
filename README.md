[![Build Status](http://runbot.odoo.com/runbot/badge/flat/1/master.svg)](http://runbot.odoo.com/runbot)
[![Tech Doc](http://img.shields.io/badge/master-docs-875A7B.svg?style=flat&colorA=8F8F8F)](http://www.odoo.com/documentation/master)
[![Help](http://img.shields.io/badge/master-help-875A7B.svg?style=flat&colorA=8F8F8F)](https://www.odoo.com/forum/help-1)
[![Nightly Builds](http://img.shields.io/badge/master-nightly-875A7B.svg?style=flat&colorA=8F8F8F)](http://nightly.odoo.com/)







**https://github.com/narGoThic/MarmaraChain bu makalede ki anlatımı yaparak Local bilgisayarınıza ubuntu kurabilirsiniz.**

**ubuntu kurulumu yaptıktan sonra. VirtualBox açıp. kurduğunuz sunucuyu başlattıktan sonra.**

**masaüstün de boş bir yerde sağ tıklayarak. Uç birim(terminal) açığ aşağıdaki komutları izleyerek. 
Odoo 12 kurulumunu gerçekleştirebilirsiniz.**

**(AZURE KURUCAKLAR BU YUKARIDA YAZANLARI DİKKATE ALMASIN)**

**STEP 1**
```
sudo apt-get update
sudo apt-get -y upgrade
sudo apt install git python3-pip build-essential wget python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less
```

**STEP 2 - Create Odoo user**

```
sudo useradd -m -d /opt/odoo12 -U -r -s /bin/bash odoo12

```

**STEP 3 - Install PostgreSQL 9.6+**
```
sudo apt install postgresql
sudo su - postgres -c "createuser -s odoo12"

```

**STEP 3,1 - Install Wkhtmltopdf** 
```
wget https://builds.wkhtmltopdf.org/0.12.1.3/wkhtmltox_0.12.1.3-1~bionic_amd64.deb
sudo apt install ./wkhtmltox_0.12.1.3-1~bionic_amd64.deb
```

**STEP 4 - Odoo kurulum ve ayarları**

```
sudo su - odoo12
git clone https://www.github.com/odoo/odoo --depth 1 --branch 12.0 /opt/odoo12/odoo
cd /opt/odoo12
python3 -m venv odoo-venv
source odoo-venv/bin/activate
pip3 install wheel
pip3 install -r odoo/requirements.txt
pip install -r odoo/requirements.txt
deactivate
mkdir /opt/odoo12/odoo-custom-addons
exit
```
**STEP 5 - odoo12.conf ayarları**
```
sudo cp /opt/odoo12/odoo/debian/odoo.conf /etc/odoo12.conf

aşağıdaki komutla odoo12.conf dosyasını açıp bir alt satırdaki kodları silip içine yapıştırıyoruz.
sudo nano /etc/odoo12.conf

alttaki kodları "my_admin_passwd" kısmını değiştirerek kendinize bir şifre belirleyip. "odoo12.conf" dosyasını kaydediyoruz (CTRL+X - YES der enterlarsanız kaydeder)
[options]
; This is the password that allows database operations:
admin_passwd = my_admin_passwd
db_host = False
db_port = False
db_user = odoo12
db_password = False
addons_path = /opt/odoo12/odoo/addons,/opt/odoo12/odoo-custom-addons

```

**STEP 6 - Odoo service ayarlama**
dosyasını açarak aşağıdaki komutları içine yazın kaydedin.(CTRL+X - Y Enter)
`sudo nano /etc/systemd/system/odoo12.service`
```
[Unit]
Description=Odoo12
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo12
PermissionsStartOnly=true
User=odoo12
Group=odoo12
ExecStart=/opt/odoo12/odoo-venv/bin/python3 /opt/odoo12/odoo/odoo-bin -c /etc/odoo12.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```

**STEP 7 - Odoo systemctl**
```
sudo systemctl daemon-reload
sudo systemctl start odoo12
sudo systemctl status odoo12
sudo systemctl enable odoo12
sudo journalctl -u odoo12

bu komutları sırasıyla girdikten sonra artık odoo ön kuruluma geçebilirsiniz
```


normal bilgisayarınızdan azure sunucu ip'nizi ve odoo portunu girerek odoo kullanabilirsiniz. 
http://sunucuip:8069
 
**giriyoruz. ve odoo açılıyor.**



**NOt : azure hesabınıza girerek "8069" portu eklemeniz gerekmektedir yoksa odoo açılmayacaktır.

-------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------
                        **BU KISIMDAN SONRASI ÖĞRENCİLER İÇİN DEĞİLDİR.!**
```

sudo nano /etc/nginx/sites-enabled/example.com

# Odoo servers
upstream odoo {
 server 127.0.0.1:8069;
}

upstream odoochat {
 server 127.0.0.1:8072;
}

# HTTP -> HTTPS
server {
    listen 80;
    server_name www.example.com example.com;

    include snippets/letsencrypt.conf;
    return 301 https://example.com$request_uri;
}

# WWW -> NON WWW
server {
    listen 443 ssl http2;
    server_name www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    include snippets/ssl.conf;

    return 301 https://example.com$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    proxy_read_timeout 720s;
    proxy_connect_timeout 720s;
    proxy_send_timeout 720s;

    # Proxy headers
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    # SSL parameters
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    include snippets/ssl.conf;

    # log files
    access_log /var/log/nginx/odoo.access.log;
    error_log /var/log/nginx/odoo.error.log;

    # Handle longpoll requests
    location /longpolling {
        proxy_pass http://odoochat;
    }

    # Handle / requests
    location / {
       proxy_redirect off;
       proxy_pass http://odoo;
    }

    # Cache static files
    location ~* /web/static/ {
        proxy_cache_valid 200 90m;
        proxy_buffering on;
        expires 864000;
        proxy_pass http://odoo;
    }

    # Gzip
    gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
    gzip on;
}
```

```
sudo systemctl restart nginx
proxy_mode = True
sudo systemctl restart odoo12
xmlrpc_interface = 127.0.0.1
netrpc_interface = 127.0.0.1
sudo systemctl restart odoo12
```

```
/etc/odoo12.conf
limit_memory_hard = 2684354560
limit_memory_soft = 2147483648
limit_request = 8192
limit_time_cpu = 600
limit_time_real = 1200
max_cron_threads = 1
workers = 5
```
sudo systemctl restart odoo12

































Odoo
----

Odoo is a suite of web based open source business apps.

The main Odoo Apps include an <a href="https://www.odoo.com/page/crm">Open Source CRM</a>,
<a href="https://www.odoo.com/page/website-builder">Website Builder</a>,
<a href="https://www.odoo.com/page/e-commerce">eCommerce</a>,
<a href="https://www.odoo.com/page/warehouse">Warehouse Management</a>,
<a href="https://www.odoo.com/page/project-management">Project Management</a>,
<a href="https://www.odoo.com/page/accounting">Billing &amp; Accounting</a>,
<a href="https://www.odoo.com/page/point-of-sale">Point of Sale</a>,
<a href="https://www.odoo.com/page/employees">Human Resources</a>,
<a href="https://www.odoo.com/page/lead-automation">Marketing</a>,
<a href="https://www.odoo.com/page/manufacturing">Manufacturing</a>,
<a href="https://www.odoo.com/#apps">...</a>

Odoo Apps can be used as stand-alone applications, but they also integrate seamlessly so you get
a full-featured <a href="https://www.odoo.com">Open Source ERP</a> when you install several Apps.


Getting started with Odoo
-------------------------
For a standard installation please follow the <a href="https://www.odoo.com/documentation/master/setup/install.html">Setup instructions</a>
from the documentation.

To learn the software, we recommend the <a href="https://www.odoo.com/slides">Odoo eLearning</a>, or <a href="https://www.odoo.com/page/scale-up-business-game">Scale-up</a>, the <a href="https://www.odoo.com/page/scale-up-business-game">business game</a>. Developers can start with <a href="https://www.odoo.com/documentation/master/tutorials.html">the developer tutorials</a>
