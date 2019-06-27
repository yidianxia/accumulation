### Nginx

> Nginx是一款轻量级的HTTP服务器，采用事件驱动的异步非阻塞处理方式框架，这让其具有极好的IO性能，时常用于服务端的反向代理和负载均衡。

- 支持海量高并发
- 内存消耗小
- 免费使用可以商业化
- 配置文件简单

**先看配置说明**

```nginx
user nobody;
#启动进程,通常设置成和cpu的数量相等
worker_processes  1;
 
#全局错误日志及PID文件
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
 
#pid        logs/nginx.pid;
 
#工作模式及连接数上限
events {
    #epoll是多路复用IO(I/O Multiplexing)中的一种方式,
    #仅用于linux2.6以上内核,可以大大提高nginx的性能
    use   epoll; 
 
    #单个后台worker process进程的最大并发链接数    
    worker_connections  1024;
 
    # 并发总数是 worker_processes 和 worker_connections 的乘积
    # 即 max_clients = worker_processes * worker_connections
    # 在设置了反向代理的情况下，max_clients = worker_processes * worker_connections / 4  为什么
    # 为什么上面反向代理要除以4，应该说是一个经验值
    # 根据以上条件，正常情况下的Nginx Server可以应付的最大连接数为：4 * 8000 = 32000
    # worker_connections 值的设置跟物理内存大小有关
    # 因为并发受IO约束，max_clients的值须小于系统可以打开的最大文件数
    # 而系统可以打开的最大文件数和内存大小成正比，一般1GB内存的机器上可以打开的文件数大约是10万左右
    # 我们来看看360M内存的VPS可以打开的文件句柄数是多少：
    # $ cat /proc/sys/fs/file-max
    # 输出 34336
    # 32000 < 34336，即并发连接总数小于系统可以打开的文件句柄总数，这样就在操作系统可以承受的范围之内
    # 所以，worker_connections 的值需根据 worker_processes 进程数目和系统可以打开的最大文件总数进行适当地进行设置
    # 使得并发总数小于操作系统可以打开的最大文件数目
    # 其实质也就是根据主机的物理CPU和内存进行配置
    # 当然，理论上的并发总数可能会和实际有所偏差，因为主机还有其他的工作进程需要消耗系统资源。
    # ulimit -SHn 65535
 
}
 
 
http {
    #设定mime类型,类型由mime.type文件定义
    include    mime.types;
    default_type  application/octet-stream;
    #设定日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
 
    access_log  logs/access.log  main;
 
    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，
    #对于普通应用，必须设为 on,
    #如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，
    #以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile     on;
    #tcp_nopush     on;
 
    #连接超时时间
    #keepalive_timeout  0;
    keepalive_timeout  65;
    tcp_nodelay     on;
 
    #开启gzip压缩
    gzip  on;
    gzip_disable "MSIE [1-6].";
 
    #设定请求缓冲
    client_header_buffer_size    128k;
    large_client_header_buffers  4 128k;
 
 
    #设定虚拟主机配置
    server {
        #侦听80端口
        listen    80;
        #定义使用 www.nginx.cn访问
        server_name  www.nginx.cn;
 
        #定义服务器的默认网站根目录位置
        root html;
 
        #设定本虚拟主机的访问日志
        access_log  logs/nginx.access.log  main;
 
        #默认请求
        location / {
            
            #定义首页索引文件的名称
            index index.php index.html index.htm;   
 
        }
 
        # 定义错误提示页面
        error_page   500 502 503 504 /50x.html;
        location = /50x.html {
        }
 
        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            
            #过期30天，静态文件不怎么更新，过期可以设大一点，
            #如果频繁更新，则可以设置得小一点。
            expires 30d;
        }
 
        #PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
        location ~ .php$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
 
        #禁止访问 .htxxx 文件
            location ~ /.ht {
            deny all;
        }
 
    }
}
```

**启动 nginx 服务：**

```
systemctl start nginx.service
or
nginx
```

查询服务的运行情况

```
ps aux | grep nginx
```

**停止 nginx 服务:**

- 立即停止服务。 无论进程是否在工作，都直接停止进程

```
nginx -s stop 
```

- 从容停止服务。相对温和，需要进程完成当前工作后再停止

```
nginx -s quit
```

- 杀死进程。很暴力，上面两种失败可以用

```
killall nginx
```

- systemctl 停止

```
systemctl stop nginx.service
```

**重载配置文件**

重写或者修改 nginx 配置文件之后 可以重新载入 

```
nginx -s reload
```

在默认情况下，Nginx 启动后会监听80端口，从而提供 HTTP 访问，如果80端口已经被占用则会启动失败。

可以使用`netstat -tlnp`命令查看端口号的占用情况。

**自定义错误页**

```nginx
# 通过状态码，返回指定的错误页面
error_page 500 502 503 504 /50x.html;
error_page 404 /404.html
location = /50x.html {
    root /source/error_page;
}
```

**访问控制**

直接在 default.conf 进行配置：

```nginx
location / {
	deny 123.4.56.78 
	allow 12.34.567.789
    deny all
}
# 同一个块下的两个权限指令，先出现的设置会覆盖后出现的设置（也就是谁先触发，谁起作用）。
```

**匹配规则**

```nginx
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

- `=`  表示精确匹配。只有请求的url路径与后面的字符串完全相等时，才会命中（优先级最高）。

- `^~`  表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找。

- `~`  表示该规则是使用正则定义的，区分大小写。

- `~*`  表示该规则是使用正则定义的，不区分大小写。

**基于端口号配置虚拟主机**

基于端口号来配置虚拟主机，算是 Nginx 中最简单的一种方式了。原理就是Nginx监听多个端口，根据不同的端口号，来区分不同的网站。

```nginx
server{
        listen 8080;
        server_name localhost;
        root /usr/share/nginx/html/html;
        index index.html;
}
```

**基于 IP 的虚拟主机**

基于 IP 和基于端口的配置几乎一样，只是把`server_name`选项，配置成 IP 就可以了。

```nginx
server{
        listen 80;
        server_name 12.34.56.789;
        root /usr/share/nginx/html/html;
        index index.html;
}
```

**配置以域名为划分的虚拟主机**

```nginx
server{
        listen 80;
        server_name xxx.xxx.com;
        location / {
                root /usr/share/nginx/html/html8001;
                index index.html index.htm;
        }
}
```

**反向代理**

现在的web模式基本的都是标准的CS结构，即Client端到Server端。那代理就是在Client端和Server端之间增加一个提供特定功能的服务器，这个服务器就是我们说的代理服务器。

- 正向代理

翻墙工具就是正向代理工具。它会把我们访问墙外服务器server的网页请求，代理到一个可以访问该网站的代理服务器proxy，这个代理服务器proxy把墙外服务器server上的网页内容获取，再转发给客户。

概括说：就是客户端和代理服务器可以直接互相访问，属于一个LAN（局域网）；代理对用户是**非透明**的，即用户需要自己操作或者感知得到自己的请求被发送到代理服务器；代理服务器通过**代理用户端的请求**来向域外服务器请求响应内容。

- 反向代理

客户端发送的请求，想要访问server服务器上的内容。但将被发送到一个代理服务器proxy，这个代理服务器将把请求代理到和自己属于同一个LAN下的内部服务器上，而用户真正想获得的内容就储存在这些内部服务器上。

好处： 安全，提供负载均衡 缓存等。

**简单的反向代理**

现在我们要访问`http://xxx.xxx.com;`然后反向代理到`www.baidu.com`这个网站。我们直接修改配置文件如下：

```nginx
server{
        listen 80;
        server_name xxx.xxx.com;
        location / {
               proxy_pass http://www.baidu.com;
        }
}
```

反向代理还有些常用的指令：

- proxy_set_header :在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息。
- proxy_connect_timeout:配置 Nginx 与后端代理服务器尝试建立连接的超时时间。
- proxy_read_timeout : 配置 Nginx 向后端服务器组发出read请求后，等待相应的超时时间。
- proxy_send_timeout：配置 Nginx 向后端服务器组发出write请求后，等待相应的超时时间。
- proxy_redirect :用于修改后端服务器返回的响应头中的 Location 和 Refresh。

**Nginx 适配 Pc 或者 mobile**

Nginx通过内置变量`$http_user_agent`，可以获取到请求客户端的userAgent，就可以用户目前处于移动端还是PC端，进而展示不同的页面给用户。

```nginx
server{
        listen 80;
        server_name xxx.xxx.com;
        location / {
         root /usr/share/nginx/pc;
         if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
            root /usr/share/nginx/mobile;
            # 并没有列举全
         }
         index index.html;
        }
}
```

**配置 gzip**

我们可以用打包工具(webpack, rollup)，将代码进行压缩，以缩小代码体积。 开启Nginx Gzip压缩功能。需要注意的是 Gzip 压缩功能需要浏览器跟服务器都支持，即服务器压缩，浏览器解析。

- 查看浏览器支持情况，确定 *请求头* 中的`Accept-Encoding`字段
- 确定浏览器支持，我们就可以在Nginx中配置

```nginx
server {
    # 开启gzip 压缩
    gzip on;
    # 设置gzip所需的http协议最低版本 （HTTP/1.1, HTTP/1.0）
    gzip_http_version 1.1;
    # 设置压缩级别，压缩级别越高压缩时间越长  （1-9）
    gzip_comp_level 4;
    # 设置压缩的最小字节数， 页面Content-Length获取
    gzip_min_length 1000;
    # 设置压缩文件的类型  （text/html)
    gzip_types text/plain application/javascript text/css;
}
# gzip : 该指令用于开启或 关闭 gzip 模块。
# gzip_buffers : 设置系统获取几个单位的缓存用于存储 gzip 的压缩结果数据流。
# gzip_comp_level : gzip 压缩比，压缩级别是1-9，1的压缩级别最低，9的压缩级别最高。压缩级别越高压缩率越大，压缩时间越长。
# gzip_disable : 可以通过该指令对一些特定的 User-Agent 不使用压缩功能。
# gzip_min_length:设置允许压缩的页面最小字节数，页面字节数从相应消息头的 Content-length 中进行获取。
# gzip_http_version：识别HTTP协议版本，其值可以是1.1.或1.0.
# gzip_proxied : 用于设置启用或禁用从代理服务器上收到相应内容gzip压缩。
# gzip_vary : 用于在响应消息头中添加 Vary：Accept-Encoding,使代理服务器根据请求头中的 Accept-Encoding 识别是否启用gzip压缩。
```

- 查看配置是否生效，查看 *响应头* 中的`Content-Encoding`字段，值为 `gzip`

==如果已经通过webpack 或者 rollup 打包出来了 gz 文件， nginx 配置 gzip_static on；开启代理静态gzip文件就好了，就不需要服务器再进行压缩，节省服务器资源==

**负载均衡**

> 负载均衡是Nginx 比较常用的一个功能，可优化资源利用率，最大化吞吐量，减少延迟，确保容错配置，将流量分配到多个后端服务器。

```
Syntax:	upstream name { ... }
Default: —
Context: stream
```

这里举出常用的几种策略

- 轮询（默认），请求过来后，Nginx 随机分配流量到任一服务器

```
upstream backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}
```

- `weight=number` 设置服务器的权重，默认为1，权重大的会被优先分配

```
upstream backend {
    server 127.0.0.1:3000 weight=2;
    server 127.0.0.1:3001 weight=1;
}
```

- `backup` 标记为备份服务器。当主服务器不可用时，将传递与备份服务器的连接。

```
upstream backend {
    server 127.0.0.1:3000 backup;
    server 127.0.0.1:3001;
}
```

- `ip_hash` 保持会话，保证同一客户端始终访问一台服务器。

```
upstream backend {
    ip_hash;  
    server 127.0.0.1:3000 backup;
    server 127.0.0.1:3001;
}
```

- `least_conn` 优先分配最少连接数的服务器，避免服务器超载请求过多。

```
upstream backend {
    least_conn;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}
```

当我们需要代理一个集群时候可以通过下面这种方式实现

```
http {

    upstream backend {
        server 127.0.0.1:3000;
        server 127.0.0.1:3001;
    }

    ...
    server {
        listen      9000;
        server_name localhost;
        
        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Scheme $scheme;
            
            proxy_pass backend; 
        }
    }
}
```


