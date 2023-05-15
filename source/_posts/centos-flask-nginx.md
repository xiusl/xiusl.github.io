---
title: centos-flask-nginx
date: 2019-01-21 12:07:19
tags: [centos, flask]
categories: [host]
---
在 centos7 上使用 nginx + uwsgi 部署 flask 应用
<!-- more -->
安装 `zsh` 和 `ohmyzsh`
```sh
yum install -y zsh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

安装一些需要的库
```sh
 yum groupinstall -y "Development tools"

 yum install -y zlib-devel bzip2-devel pcre-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel libffi-devel
```
安装 `Python3`，这里编译安装，也可以使用 `yum` 来安装
```sh
 wget http://python.org/ftp/python/3.7.1/Python-3.7.1.tar.xz

 xz -d Python3.7.1.tar.xz

 tar -xvf Python3.7.1.tar

 cd Python3.7.1

 ./configure --prefix=/usr/local

 make && make install
```
建立软连接
```sh
ln -s /usr/local/bin/python3 /usr/bin/python
# ln: 无法创建符号链接"/usr/bin/python": 文件已存在
# rm /usr/bin/python
```
修复 yum、firewall-cmd 等
```sh
vim /usr/bin/yum
#! /usr/bin/python => /usr/bin/python2
vim /usr/libexec/urlgrabber-ext-down
#! /usr/bin/python => /usr/bin/python2
vim vim /usr/bin/firewall-cmd
#!/usr/bin/python -Es => #!/usr/bin/python2 -Es
```
安装 `uwsgi`
```sh
pip3 install uwsgi
```
安装 `Python` 虚拟环境 [virtualenv](https://virtualenv.pypa.io/en/stable/installation/)
```sh
pip3 install virtualenv
```
根目录下创建工作文件夹，虚拟环境
```sh
cd /
mkdir project
cd project
virtualenv env --python=/youpython3path/python3
source env/bin/activate
```
安装 `flask`
```sh
echo flask > requirements.txt
pip install -r requirements.txt
```
创建一个简单flask 应用 `vim manage.py`
```py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "Hello"
```
创建 `uwsgi` 配置文件 [uwsgi.ini](https://uwsgi-docs.readthedocs.io/en/latest/Configuration.html)
```ini
[uwsgi]
socket = 127.0.0.1:9090
master = true
vhost = true
no-stie = true
workers = 2
reload-mercy = 10
vacuum = true
max-requests = 1000
limit-as = 2048
buffer-size = 30000
daemonize = /var/log/uwsgi.log

chdir = /project
virtualenv = /project/env
wsgi-file = manage.py
callable = app
pidfile = /run/uwsgi.pid
```
安装 `nginx`
```sh
yum install -y nginx
```
修改 nginx 配置 `vim /etc/nginx/nginx.conf`
```conf
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  _;
#    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    	uwsgi_pass  127.0.0.1:9090;
        include uwsgi_params;
    }

#    error_page 404 /404.html;
#        location = /40x.html {
#    }

#    error_page 500 502 503 504 /50x.html;
#        location = /50x.html {
#    }
}
```
测试 `nginx` 配置是否正确 `nginx -t`
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
`nginx` 一些操作
```sh
nginx
nginx -s reload
```
开启 `uwsgi`
```sh
uwsgi -i /etc/uwsgi.ini
```
重启 `uwsgi`
```sh
uwsgi --reload run/uwsgi.pid
```
