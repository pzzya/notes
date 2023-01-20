# [Nginx](http://nginx.org/en/)

> Nginx("engine x")是一款是由俄罗斯的程序设计师Igor Sysoev所开发高性能的 Web和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 代理服务器。

### 下载解压

```sh
cd ~
# nginx官网  http://nginx.org/en/
# 下载
wget http://nginx.org/download/nginx-1.23.3.tar.gz

# 解压
tar -zxvf ./nginx-1.23.3.tar.gz
cd nginx-1.23.3

```

### 编译安装

```sh
# 安装C编译器
sudo yum install -y gcc

# 安装pcre库
sudo yum install -y pcre pcre-devel

# 安装zlib
sudo yum install -y zlib zlib-devel

# 使用prefix选项指定安装的目录
sudo ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module

# 编译 
# 若编译报错：SSL modules require the OpenSSL library 是缺少 openssl 环境，需要手动安装
# CentOS 👉  yum -y install openssl openssl-devel
# Ubuntu 👉  apt -y install openssl openssl-devel
sudo make 

# 安装
sudo make install
```

### 操作命令

```sh
cd /usr/local/nginx/sbin

# 启动nginx可执行
./nginx

# 补充Nginx命令
# 快速停止
./nginx -s stop

# 完成已接受的请求后，停止
./nginx -s quit 

# 重新加载配置文件
./nginx -s reload

# 检查nginx配置是否正确
./nginx -t 
```

### 注册服务

```sh
# 使用 vim 编辑保存 nginx.service
vim /usr/lib/systemd/system/nginx.service

[Unit] 
Description=nginx
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
ExecQuit=/usr/local/nginx/sbin/nginx -s quit 
PrivateTmp=true
   
[Install]   
WantedBy=multi-user.target


# 重新加载系统服务
systemctl daemon-reload

# 设置开机启动
systemctl enable nginx.service

# 使用系统服务操作nginx
# 启动
systemctl start nginx 

# 重新加载配置文件
# 修改nginx配置文件后,需要重启nginx生效
systemctl reload nginx 

# 停止
systemctl stop nginx

# 查看状态
systemctl status nginx 
```

# [Let’s Encrypt](https://letsencrypt.org/zh-cn/) & [certbot](https://certbot.eff.org/)

> Let’s Encrypt 是一个为 2.25 亿个网站提供 TLS 证书的非盈利性证书颁发机构。certbot 是一个免费的开源软件工具，用于在手动管理的网站上自动使用Let's Encrypt证书来启用HTTPS。

### 下载安装

```sh
# 安装epel
sudo yum install epel-release

# 安装snapd。
yum install snapd

# 启用snapd.socket。
systemctl enable --now snapd.socket

# 创建/var/lib/snapd/snap和/snap之间的链接。
ln -s /var/lib/snapd/snap /snap

# 退出账号并重新登录，或者重启系统，确保snap启用。

# 将snap更新至最新版本。
sudo snap install core
sudo snap refresh core

# 使用snap安装certbot
snap install --classic certbot

# 创建/snap/bin/certbot的软链接，方便certbot命令的使用。
ln -s /snap/bin/certbot /usr/bin/certbot
```

### 生成证书

```sh
sudo certbot certonly  -d *.example.com -d example.com --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

|参数|说明|
|--|--|
|certonly|表示安装模式 certbot 有安装模式和验证模式两种类型的插件|
|--manual|表示手动安装插件,certbot有很多插件.不同的插件都可以申请证书,用户可以根据需要自行选择|
|-d|为哪些主机申请证书,支持通配符|
|--preferred-challenges dns|使用DNS方式校验域名所有权|
|--server|Let’s Encrypt ACME v2 版本使用的服务器不同于v1,需要指定|

### 略...

> 接下来会输入一些指令,如果没变化的 根据提示输入自己的信息或选择

### 需要注意的地方

> 这里需要在域名解析里添加一条 `_acme-challenge` 的 `TXT` 记录,来验证域名所有权. ( 这一步被要求 根据你注册证书时填写的域名数量添加多条内容不同的`_acme-challeng` 记录 ) 在确保记录生效后回车执行.

```sh
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.you.cn with the following value:

RYtObhDvEcXewZckknNQkBKIkvwIlbb4PNRel74LNwU

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
Waiting for verification...
Cleaning up challenges
```

### 输出结果

> 输出结果会有证书所在的位置,记下证书的路径

```sh
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2019-02-27. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

### 证书续签

> 证书在到期前30天才会续签成功,但为了确保证书在运行过程中不过期,官方建议每天自动执行续签两次

```sh
# 编辑定时任务
crontab -e

0 */12 * * * certbot renew --quiet --renew-hook "sudo systemctl reload nginx"

```

<br/>

# nginx.conf

```sh
sudo vim /etc/local/nginx/conf/nginx.conf
```

```sh
#user  nobody;
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include mime.types;
    default_type application/octet-stream;

    sendfile on; #文件不存内存

    keepalive_timeout 65;

    # GZIP 相关
    #
    gzip on; #是否开启gzip模块 on表示开启 off表示关闭
    gzip_buffers 4 16k; #设置压缩所需要的缓冲区大小
    gzip_comp_level 6; #压缩级别1-9，数字越大压缩的越好，也越占用CPU时间
    gzip_min_length 100k; #设置允许压缩的最小字节
    gzip_http_version 1.1; #设置压缩http协议的版本,默认是1.1
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript; #设置压缩的文件类型
    gzip_vary on; #加上http头信息Vary: Accept-Encoding给后端代理服务器识别是否启用 gzip 压缩

    # SSL性能调优
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES256-SHA384:AES256-SHA256:RC4:HIGH:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!AESGCM;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;


    # www.example.com
    server {
        listen 443 ssl;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
        server_name www.pzzya.com ;

        autoindex on;

        charset utf-8;

        location / {
            root /home/www/www;
            index index.html;
        }

        # 404错误重定向
        # error_page  404              /404.html;

        # 500错误重定向
        #
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root html;
        }
    }

    # server.example.com
    server {
        listen 443 ssl;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

        server_name server.pzzya.com;
        location / {
            proxy_pass http://127.0.0.1:3000/;
        }
    }
    
    # http 转发到 https
    server {
        listen 80;
        server_name example.com *.example.com;
        location / {
            rewrite ^(.*)$ https://$host$1 permanent;
        }
    }
}
```

# 参考链接

[尚硅谷2022版Nginx教程(亿级流量nginx架构设计)](https://www.bilibili.com/read/cv17681652)

[Nginx 编译报错：SSL modules require the OpenSSL library](https://blog.csdn.net/weixin_44364444/article/details/117427650)

[解决nginx: \[emerg\] the “ssl“ parameter requires ngx_http_ssl_module in /usr/local/nginx的问题](https://blog.csdn.net/guo_qiangqiang/article/details/95622649)

[Let's Encrypt 安装配置教程，免费的 SSL 证书](https://zhuanlan.zhihu.com/p/196638669)

[certbot-auto不再支持所有的操作系统,新的ssl证书方法](https://blog.csdn.net/qq_27346503/article/details/114188857)

***

本文部分内容来源网络文章, 仅为个人记录学习, 分享知识, 如有侵权 请联系作者删除 📫pzzyaa@outlook.com
