## 前言

这里主要是Nginx的配置，使Nginx转发请求到PHP-FPM。

## nginx.conf
首页要修改的就是nginx的配置文件，这个文件一般在/etc/nginx目录下。

下面就是这个文件的内容，主要是在#Virtual Host Configs下面引入虚拟主机的配置文件目录,`include /etc/nginx/sites-enabled/*` ，设置目录后就会加载sites-enabled目录下的所有配置文件，不过你也可以限制一下配置文件的格式，比如`*.conf`。

注意这里引入include的操作需要在server块尾部加入，这点需要注意

```conf
user vagrant;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        log_format main '$remote_addr - $remote_user [$time_local] '
                       '"$request" $status $bytes_sent '
                       '"$http_referer" "$http_user_agent" "$gzip_ratio"';

        access_log /var/log/nginx/access.log main;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
```

## 虚拟主机配置
上面我们在nginx.conf配置文件中引入了虚拟主机的配置目录`include /etc/nginx/sites-enabled/*`,就可以在这个目录下新建配置文件。

比如我们现在本地开发的域名为tp.test,我们就创建一个名字为`tp.test`的配置文件。

文件具体内容如下：
```conf
server {
    listen 80;
    listen 443 ssl http2;
    # 虚拟域名
    server_name tp.test;
    # 项目的路径
    root "/home/vagrant/Code/tp/public";
    # 默认访问的文件
    index index.html index.htm index.php;

    charset utf-8;

    location / {
    	# 尝试加上index.php
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/tp.test-error.log error;

    sendfile off;

    client_max_body_size 100m;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        # fastcgi的配置
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
    }

    location ~ /\.ht {
        deny all;
    }

    ssl_certificate     /etc/nginx/ssl/tp.test.crt;
    ssl_certificate_key /etc/nginx/ssl/tp.test.key;
}

```
这个配置文件的内容还是比较多的，具体就涉及到nginx的知识了，可以看下这块的知识。

## 重启nginx
修改了nginx的文件最好平滑的启动nginx，让nginx去加载虚拟配置文件。

启动前我们可以测试一下配置文件是否正确`sudo nginx -t -c /etc/nginx/nginx.conf`

上面的命令执行后没有问题了，我们可以使用命令平滑重启nginx `sudo nginx -s reload`


## nginx不支持pathinfo
如果nginx不支持pathinfo模式，可以添加如下配置。
```conf
	server {
	......
    location / {
    	# 尝试加上index.php
        if (!-e $request_filename){
        	rewrit ^/index.php(.*)$ /index.php?s=$1 last;
        	rewrit ^(.*)$ /index.php?s=$1 last;
        	break;
        }
    }
    ......
  }
```

## 设置access和error日志
对于每个虚拟主机我们都可以设置不同的访问日志。在server模块中我们就可以设置
aceess和error的错误日志。

```conf
server {
	....
	access_log /var/log/nginx/tp.test-access.log  main
	error_log /var/log/nginx/tp.test-error.log  error
	....
}
```

对于像favicon.ico不记录访问日志，可以如下设置，具体的语法可以看教程。
```conf
server{
	location = /favicon.ico { access_log off; log_not_found off; }
	location = /robots.txt  { access_log off; log_not_found off; }
}
```