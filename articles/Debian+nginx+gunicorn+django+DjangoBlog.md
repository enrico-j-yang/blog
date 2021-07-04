#更新系统
>sudo apt-get update
>sudo apt-get upgrade

#安装数据库
>sudo apt-get install default-mysql-server

##创建数据库
>sudo mysql -u root -p

```
CREATE DATABASE database_name;
```
##创建用户
```
CREATE USER username;
```
##设置权限
```
GRANT ALL PRIVILEGES ON database_name.*. TO 'username'@'localhost' IDENTIFIED BY 'password';
```
##刷新退出
```
FLUSH PRIVILEGES;
EXIT;
```


#创建python虚拟环境
>sudo apt install python3-venv

##变量
* dir_name – Name of the directory
* venv_name – Name of the virtual environment.

```
mkdir dir_name
 
cd dir_name
```
```
python3 -m venv venv_name
 
source venv_name/bin/activate
```
#拉取代码
git clone git@github.com:liangliangyy/DjangoBlog.git

##修改DjangoBlog/settings.py
```
ALLOWED_HOSTS = ['*', '127.0.0.1', 'example.com', 'your_ip']
.
.
.

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.environ.get('DJANGO_MYSQL_DATABASE') or 'database_name',
        'USER': os.environ.get('DJANGO_MYSQL_USER') or 'username',
        'PASSWORD': os.environ.get('DJANGO_MYSQL_PASSWORD') or 'password',
        'HOST': os.environ.get('DJANGO_MYSQL_HOST') or '127.0.0.1',
        'PORT': int(
            os.environ.get('DJANGO_MYSQL_PORT') or 3306),
        'OPTIONS': {
            'charset': 'utf8mb4'},
    }}
```
##安装python模组
>pip3 install -Ur requirements.txt

##初始化django数据
>python3 manage.py makemigrations
>python3 manage.py migrate

##创建超级用户
终端下执行:

>python3 manage.py createsuperuser

##创建测试数据
终端下执行:

>python3 manage.py create_testdata

##收集静态文件
终端下执行:  

>python3 manage.py collectstatic --noinput
>python3 manage.py compress --force

##开始运行：
执行： python3 manage.py runserver

浏览器打开: http://127.0.0.1:8000/ 就可以看到效果了。

##检查部署
>python manage.py check --deploy

#Gunicorn 配置
>sudo apt-get install gunicorn3

##测试一下部署
这里的wsgi名一定要注意大小写
>gunicorn3 --bind 0.0.0.0:8000 DjangoBlog.wsgi

浏览器再打开: http://127.0.0.1:8000/ 看看

好了，crtl + c 中止运行Gunicorn 
##退出python虚拟环境
> deactivate 

Gunicorn使用.sock文件来与其他进程沟通
> sudo mkdir /var/log/gunicorn

然后我们把它做成系统服务
>sudo nano /etc/systemd/system/gunicorn.service

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=pi
Group=www-data
WorkingDirectory=/home/pi/Documents/DjangoBlog
ExecStart=sudo /usr/bin/gunicorn3 --access-logfile - --workers 3 --bind unix:/var/log/gunicorn/djangoblog.sock DjangoBlog.wsgi:application

[Install]
WantedBy=multi-user.target
```
做成自启动
>sudo systemctl start gunicorn
>sudo systemctl enable gunicorn

看看服务设置好了没
>sudo systemctl status gunicorn

```
● gunicorn.service - gunicorn daemon
   Loaded: loaded (/etc/systemd/system/gunicorn.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2020-04-25 13:31:07 UTC; 2h 5min ago
 Main PID: 19668 (gunicorn)
```

如果有错误，可以这样看原因
>sudo journalctl -u gunicorn

如果有修改，要重新加载一下
>sudo systemctl daemon-reload
>sudo systemctl restart gunicorn

#设置Nginx
>sudo apt-get install nginx

修改项目的settings.py
>STATIC_ROOT = '/var/www/static'

输出静态文件
>python3 manage.py collectstatic

##变量

>project_name– Name of the project or domain.

>sudo nano /etc/nginx/sites-available/project_name

然后加个文件 
##变量说明

>server_domain_or_IP -域名或者ip地址.
>static_root_path : settings.py路径
>/path/to/staticfiles: 按此指引应该是/var/www

```
upstream hello_app_server {
    server unix:/var/log/gunicorn/djangoblog.sock fail_timeout=0s;
}


server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location  /static/ {
        root /path/to/staticfiles;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/log/gunicorn/djangoblog.sock;
    }
}
```

删除/etc/nginx/sites-enabled/default
做个软连接
>sudo ln -s /etc/nginx/sites-available/djangoblog /etc/nginx/sites-enabled

测试一下
>sudo nginx -t

启动nginx服务
>sudo systemctl restart nginx

#设置防火墙
>sudo apt-get install ufw
>sudo ufw delete allow 8000
>sudo ufw allow 'Nginx Full'

再打开
>http://domain_name_or_server_IP


如果有错，查看nginx日志
>sudo tail -F /var/log/nginx/error.log

没有错误的话完成了

#参考资料
* [How To Deploy Django App with Nginx, Gunicorn, PostgreSQL and Let’s Encrypt SSL on Ubuntu](https://djangocentral.com/deploy-django-with-nginx-gunicorn-postgresql-and-lets-encrypt-ssl-on-ubuntu/)
