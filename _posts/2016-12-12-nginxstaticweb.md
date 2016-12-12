---
layout    : post
title     : Nginx配置静态web服务器
date      : 2016-12-12
author    : GSM
categories: blog             
tags      :
image     : /assets/article_images/NGINX_logo_rgb-01.png
elapse    :
---
_ 「Nginx」_

由于Nginx具有强悍的高并发高负载能力，因此一般会作为前端的服务器直接向客户端提供静态文件服务。

既然要使用Nginx配置一个静态web服务器，那么首先要厘清：静态服务器都应该实现什么功能。

早在第一个网页浏览器Mosaic时期，web服务器一开始就是一个静态web服务器。当用户向静态web服务器发送请求后，服务器返回静态网页（static web page）。所谓静态网页，可以理解为用户所得即服务器存储文件。因为无交互，用户从浏览器看到的都是同样的内容。从技术角度，静态网页都是标准的HTML，可以包括HTML标记，文本，图片，java小程序，但是不包含任何服务器端处理。静态网页放置到web服务器后便不发生任何更改。静态网页是保存在服务器上的文件，每个网页都是一个独立的文件。（目前通过Github制作的blog和book都是这种静态网页。）  
![](/assets/article_images/techarticles/nginxframework.png)  

所以可以简单理解为静态web服务器其实是：用户通过HTTP协议与某一台指定主机通信，该主机具有HTTP通信+文件系统功能。当主机收到用户的uri请求，根据uri 找到存储相应路径的文件资源(html)返回给用户，最终在浏览器中显示出来。

Nginx在这里充当的角色是：监听HTTP 80（HTTPS 443）端口，当来了一个request时候，解析出servername 与 uri, 转发到对应的location（主机文件路径）中，即请求转发。而我们知道Nginx是由多个耦合度极地的模块组成，Nginx的功能由它所包含的模块决定。静态Web服务器所包含的主要功能由`ngx_http_core_module`模块实现，这个模块默认已经编译进了nginx中。所以不需要我们额外指定configure文件。下面的重点是如何配置`ngx_http_core_module`模块的配置项，从而实现web静态服务器功能。

下面进入实验部分，通过配置nginx.conf，来实现一个静态web服务器。（windows OS）
#### 　1、 首先创建一个静态资源 midnigntInParis.html 及准备资源目录

![](/assets/article_images/techarticles/nginxstatic1.png)  

log 目录存放nginx access.log 和error.log

website 存放静态资源html文件

本实验存放一个默认的首页index.html 以及一个movie/midnightinparis.html静态资源

![](/assets/article_images/techarticles/nginxstatic2.png)  
![](/assets/article_images/techarticles/nginxstatic3.png)  

#### 　2、接着配置nginx.conf 附注释
```
worker_processes  1; 
#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen 80;
        server_name midnightInParis.com ## 设置servername. 在开始处理一个HTTP请求时，Nginx会取出一个header头中的Host, 与每个server中的server_name比较。以此决定那个server块处理这个请求。
        
        access_log /home/gsm/ngixdev/tmp/log/website/access.log;   ## 设置访问日志
        error_log /home/gsm/ngixdev/tmp/website/error.log;  ## 设置错误日志

        ##  根据用户请求的URI来匹配location,如果可以匹配则选择location{}块中的配置来处理用户请求
location / {
            root /home/gsm/ngixdev/tmp/website; ## 设置根资源路径
            index midnightInParis.html midnightInParis.htm; ## 默认首页路径
        }
        error_page 404 /404.html;
        
        # redirect server error pages to the static page /50x.html
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
               root html;
        }
    }
}
```

#### 3. 启动nginx(如果已经启动了nginx，需要关闭并重启)
#### 4. 绑定host
gsm.com 127.0.0.1
(注: 配置host是为了让本机发送gsm.com请求的时候，指定本机127.0.0.1来处理。这样才能被nginx的80端口监听到。)
#### 5. 在浏览器输入 gsm.com
进入默认index页面
![](/assets/article_images/techarticles/nginxstatic4.png)  
#### 6. 输入指定uri /movie/midnightInParis.html
![](/assets/article_images/techarticles/nginxstatic5.png)  
总结：通过简单的nginx.conf配置，就可以实现静态web服务器的基本功能。当然这只是静态web服务器的低配版本，还可以通过其他配置项来提高服务器的性能，这里不一一赘述。

###参考
1. https://www.nginx.com
2. 深入理解Nginx  陶辉[著] 出版社: 机械工业出版社