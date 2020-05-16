---
title: "Nginx使用指南"
date: 2020-05-15 14:00:00 +0800
category: Nginx
tags: [Nginx, 使用指南]
excerpt: 本文主要主要说明Nginx的概念和基本使用方法。
---

# Nginx 使用指南

> Nginx (engine x) 是一个高性能的 HTTP 和反向代理 Web 服务器，同时也提供了 IMAP/POP3/SMTP 服务。

## docker下安装Nginx

### 1、取最新版的 Nginx 镜像

```shell
$docker pull nginx:latest
```
### 2、查看本地镜像

```shell script
$docker images
````

### 3、本地配置

主要配置文件说明：

| 文件名      | 说明                                |
| ----------- | ----------------------------------- |
| nginx.conf  | Nginx 的基本配置文件                |
| mime.types  | MIME 类型关联的扩展文件             |
| astcgi.conf | 与 fastcgi 相关的配置               |
| proxy.conf  | 与 proxy 相关的配置                 |
| sites.conf  | 配置 Nginx 提供的网站，包括虚拟主机 |

- **创建挂载目录**

```shell script
$mkdir -p /data/nginx/{conf,conf.d,html,logs}
```

- **编写nginx.conf配置文件，并放在/data/nginx/conf文件夹中**

```shell
#定义 Nginx 运行的用户和用户组,默认由 nobody 账号运行, windows 下面可以注释掉。
user  nobody; 
 
#nginx进程数，建议设置为等于CPU总核心数。可以和worker_cpu_affinity配合
worker_processes  1; 
 
#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
 
#进程文件，window下可以注释掉
#pid        logs/nginx.pid;
 
# 一个nginx进程打开的最多文件描述符(句柄)数目，理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除，
# 但是nginx分配请求并不均匀，所以建议与ulimit -n的值保持一致。
worker_rlimit_nofile 65535;
 
#工作模式与连接数上限
events {
    # 参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; 
    # epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型。
   #use epoll;
   #connections 20000;  # 每个进程允许的最多连接数
   # 单个进程最大连接数（最大连接数=连接数*进程数）该值受系统进程最大打开文件数限制，需要使用命令ulimit -n 查看当前设置
   worker_connections 65535;
}
 
#设定http服务器
http {
    #文件扩展名与文件类型映射表
    #include 是个主模块指令，可以将配置文件拆分并引用，可以减少主配置文件的复杂度
    include       mime.types;
    #默认文件类型
    default_type  application/octet-stream;
    #charset utf-8; #默认编码
 
    #定义虚拟主机日志的格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
    #定义虚拟主机访问日志
    #access_log  logs/access.log  main;
 
    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    sendfile        on;
    #autoindex on; #开启目录列表访问，合适下载服务器，默认关闭。
 
    #防止网络阻塞
    #tcp_nopush     on;
 
    #长连接超时时间，单位是秒，默认为0
    keepalive_timeout  65;
 
    # gzip压缩功能设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k; #最小压缩文件大小
    gzip_buffers    4 16k; #压缩缓冲区
    gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 6; #压缩等级
    #压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
    gzip_vary on; //和http头有关系，加个vary头，给代理服务器用的，有的浏览器支持压缩，有的不支持，所以避免浪费不支持的也压缩，所以根据客户端的HTTP头来判断，是否需要压缩
    #limit_zone crawler $binary_remote_addr 10m; #开启限制IP连接数的时候需要使用
 
    # http_proxy服务全局设置
    client_max_body_size   10m;
    client_body_buffer_size   128k;
    proxy_connect_timeout   75;
    proxy_send_timeout   75;
    proxy_read_timeout   75;
    proxy_buffer_size   4k;
    proxy_buffers   4 32k;
    proxy_busy_buffers_size   64k;
    proxy_temp_file_write_size  64k;
    proxy_temp_path   /usr/local/nginx/proxy_temp 1 2;
 
   # 设定负载均衡后台服务器列表 
    upstream  backend.com  { 
        #ip_hash; # 指定支持的调度算法
        # upstream 的负载均衡，weight 是权重，可以根据机器配置定义权重。weigth 参数表示权值，权值越高被分配到的几率越大。
        server   192.168.10.100:8080 max_fails=2 fail_timeout=30s ;  
        server   192.168.10.101:8080 max_fails=2 fail_timeout=30s ;  
    }
 
    #虚拟主机的配置
    server {
        #监听端口
        listen       80;
        #域名可以有多个，用空格隔开
        server_name  localhost;
        # Server Side Include，通常称为服务器端嵌入
        #ssi on;
        #默认编码
        #charset utf-8;
        #定义本虚拟主机的访问日志
        #access_log  logs/host.access.log  main;
        
        # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
        location / {
            root   html;
            index  index.html index.htm;
        }
        
        #error_page  404              /404.html;
 
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
 
       # 图片缓存时间设置
       location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$ {
          expires 10d;
       }
 
       # JS和CSS缓存时间设置
       location ~ .*.(js|css)?$ {
          expires 1h;
       }
 
        #代理配置
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #location /proxy/ {
        #    proxy_pass   http://127.0.0.1;
        #}
 
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
 
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
 
    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
 
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;
 
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
 
    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
 
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
```

nginx.conf 配置文件主要分成四个部分：

| 设置项   | 说明                                                  |
| -------- | ----------------------------------------------------- |
| main     | 全局设置，影响其它部分所有设置                        |
| server   | 主机服务相关设置，主要用于指定虚拟主机域名、IP 和端口 |
| location | URL 匹配特定位置后的设置，反向代理、内容篡改相关设置  |
| upstream | 上游服务器设置，负载均衡相关配置                      |

他们之间的关系是：server 继承 main，location 继承server；upstream 既不会继承指令也不会被继承。

### 4、**运行容器**

```shell
$docker run \
  --name mynginx \
  -d -p 8089:80 \
  -v /data/nginx/html:/usr/share/nginx/html \
  -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
  -v /data/nginx/conf.d:/etc/nginx/conf.d \
  -v /data/nginx/log:/var/log/nginx \
  nginx
```

### **5、安装成功**

最后我们可以通过浏览器可以直接访问 8089 端口的 nginx 服务：http://localhost:8089

### **6、其他常用命令**

```shell
$docker restart mynginx;//重启nginx
$docker start mynginx;//启动nginx
$docker stop mynginx;//停止nginx
$docker rm mynginx;//删除nginx容器
```

