# Nginx面试题

## 请陈述 stub_status 和 sub_filter 指令的作用是什么?


Stub_status 指令：该指令用于了解 Nginx 当前状态的当前状态，如当前的活

动连接，接受和处理当前读/写/等待连接的总数

Sub_filter 指令：它用于搜索和替换响应中的内容，并快速修复陈旧的数据


## nignx配置


```
worker_processes  8;     工作进程个数

worker_connections  65535;  每个工作进程能并发处理（发起）的最大连接数（包含所有连接数）

error_log         /data/logs/nginx/error.log;  错误日志打印地址

access_log      /data/logs/nginx/access.log  进入日志打印地址

log_format  main  'remote_addr"request" ''status upstream_addr "$request_time"'; 进入日志格式

fastcgi_connect_timeout=300; #连接到后端fastcgi超时时间

fastcgi_send_timeout=300; #向fastcgi请求超时时间(这个指定值已经完成两次握手后向fastcgi传送请求的超时时间)

fastcgi_rend_timeout=300; #接收fastcgi应答超时时间，同理也是2次握手后

fastcgi_buffer_size=64k; #读取fastcgi应答第一部分需要多大缓冲区，该值表示使用1个64kb的缓冲区读取应答第一部分(应答头),可以设置为fastcgi_buffers选项缓冲区大小

fastcgi_buffers 4 64k;#指定本地需要多少和多大的缓冲区来缓冲fastcgi应答请求，假设一个php或java脚本所产生页面大小为256kb,那么会为其分配4个64kb的缓冲来缓存

fastcgi_cache TEST;#开启fastcgi缓存并为其指定为TEST名称，降低cpu负载,防止502错误发生

listen       80;                                            监听端口

server_name  rrc.test.jiedaibao.com;       允许域名

root  /data/release/rrc/web;                    项目根目录

index  index.php index.html index.htm;  访问根文件
```

## 请列举 Nginx 服务器的最佳用途。

Nginx 服务器的最佳用法是在网络上部署动态 HTTP 内容，使用 SCGI、WSGI 应用程序服务器、用于脚本的 FastCGI 处理程序。它还可以作为负载均衡器。


## 为什么要做动静分离？


**1、** Nginx是当下最热的Web容器，网站优化的重要点在于静态化网站，网站静态化的关键点则是是动静分离，动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们则根据静态资源的特点将其做缓存操作。

**2、** 让静态的资源只走静态资源服务器，动态的走动态的服务器

**3、** Nginx的静态处理能力很强，但是动态处理能力不足，因此，在企业中常用动静分离技术。

**4、** 对于静态资源比如图片，js，css等文件，我们则在反向代理服务器nginx中进行缓存。这样浏览器在请求一个静态资源时，代理服务器nginx就可以直接处理，无需将请求转发给后端服务器tomcat。

**5、** 若用户请求的动态文件，比如servlet,jsp则转发给Tomcat服务器处理，从而实现动静分离。这也是反向代理服务器的一个重要的作用。


## Nginx怎么判断别IP不可访问？


```
# 如果访问的ip地址为192.168.9.115,则返回403
if  ($remote_addr = 192.168.9.115) {
     return 403;
}
```


## nginx状态码


499：服务端处理时间过长，客户端主动关闭了连接。


## Location正则案例 


**示例：**

```
#优先级

精确匹配，根路径
location = / {
   return 400;
}

#优先级2,以某个字符串开头,以av开头的，优先匹配这里，区分大小写
location ^~ /av {
    root / data / av / ;
}

#优先级
3，区分大小写的正则匹配，匹配 / media * * * * * 路径location~ / media {
    alias / data / static / ;
}

#优先级
4，不区分大小写的正则匹配，所有的 * * * * .jpg | gif | png都走这里
location~ * .*\.(jpg | gif | png | js | css) $ {
    root / data / av / ;
}

#优先
7，通用匹配
location / {
    return 403;
}
```


## 解释 Nginx 是否支持将请求压缩到上游?


您可以使用 Nginx 模块 gunzip 将请求压缩到上游。gunzip 模块是一个过滤

器，它可以对不支持“gzip”编码方法的客户机或服务器使用“内容编

码:gzip”来解压缩响应。


## 使用“反向代理服务器”的优点是什么?


反向代理服务器可以隐藏源服务器的存在和特征。它充当互联网云和web服务器之间的中间层。这对于安全方面来说是很好的，特别是当您使用web托管服务时。


## 请解释是否有可能将 Nginx 的错误替换为 502 错误、503?


502 =错误网关

503 =服务器超载

有可能，但是您可以确保 fastcgi_intercept_errors 被设置为 ON，并使用错

误页面指令。

Location / {fastcgi_pass 127.0.01:9001;fastcgi_intercept_errors

on;error_page 502 =503/error_page.html;#…}

## Nginx 是如何实现高并发的？


如果一个 server 采用一个进程(或者线程)负责一个request的方式，那么进程数就是并发数。那么显而易见的，就是会有很多进程在等待中。等什么？最多的应该是等待网络传输。其缺点胖友应该也感觉到了，此处不述。

**思考下，Java 的 NIO 和 BIO 的对比哟。**

而 Nginx 的异步非阻塞工作方式正是利用了这点等待的时间。在需要等待的时候，这些进程就空闲出来待命了。因此表现为少数几个进程就解决了大量的并发问题。

Nginx是如何利用的呢，简单来说：同样的 4 个进程，如果采用一个进程负责一个 request 的方式，那么，同时进来 4 个 request 之后，每个进程就负责其中一个，直至会话关闭。期间，如果有第 5 个request进来了。就无法及时反应了，因为 4 个进程都没干完活呢，因此，一般有个调度进程，每当新进来了一个 request ，就新开个进程来处理。

**回想下，BIO 是不是存在酱紫的问题？嘻嘻。**

Nginx 不这样，每进来一个 request ，会有一个 worker 进程去处理。但不是全程的处理，处理到什么程度呢？处理到可能发生阻塞的地方，比如向上游（后端）服务器转发 request ，并等待请求返回。那么，这个处理的 worker 不会这么傻等着，他会在发送完请求后，注册一个事件：“如果 upstream 返回了，告诉我一声，我再接着干”。于是他就休息去了。此时，如果再有 request 进来，他就可以很快再按这种方式处理。而一旦上游服务器返回了，就会触发这个事件，worker 才会来接手，这个 request 才会接着往下走。

**这就是为什么说，Nginx 基于事件模型。**

由于 web server 的工作性质决定了每个 request 的大部份生命都是在网络传输中，实际上花费在 server 机器上的时间片不多。这是几个进程就解决高并发的秘密所在。即：

**webserver 刚好属于网络 IO 密集型应用，不算是计算密集型。**

而正如叔度所说的

**异步，非阻塞，使用 epoll ，和大量细节处的优化**

也正是 Nginx 之所以然的技术基石。


## 请解释 Nginx 如何处理 HTTP 请求。


Nginx 使用反应器模式。主事件循环等待操作系统发出准备事件的信号，这样数据就可以从套接字读取，在该实例中读取到缓冲区并进行处理。单个线程可以提供数万个并发连接。


## 为什么要做动、静分离？


在我们的软件开发中，有些请求是需要后台处理的（如：.jsp,.do等等），有些请求是不需要经过后台处理的（如：css、html、jpg、js等等），这些不需要经过后台处理的文件称为静态文件，否则动态文件。因此我们后台处理忽略静态文件，但是如果直接忽略静态文件的话，后台的请求次数就明显增多了。在我们对资源的响应速度有要求的时候，应该使用这种动静分离的策略去解决动、静分离将网站静态资源（HTML，JavaScript，CSS等）与后台应用分开部署，提高用户访问静态代码的速度，降低对后台应用访问。这里将静态资源放到nginx中，动态资源转发到[tomcat](https://www.wkcto.com/courses/tomcat.html)服务器中,毕竟Tomcat的优势是处理动态请求。

## nginx是如何实现高并发的？


一个主进程，多个工作进程，每个工作进程可以处理多个请求，每进来一个request，会有一个worker进程去处理。但不是全程的处理，处理到可能发生阻塞的地方，比如向上游（后端）服务器转发request，并等待请求返回。那么，这个处理的worker继续处理其他请求，而一旦上游服务器返回了，就会触发这个事件，worker才会来接手，这个request才会接着往下走。由于web server的工作性质决定了每个request的大部份生命都是在网络传输中，实际上花费在server机器上的时间片不多。这是几个进程就解决高并发的秘密所在。即@skoo所说的webserver刚好属于网络io密集型应用，不算是计算密集型。


## Nginx静态资源?


静态资源访问，就是存放在nginx的html页面，我们可以自己编写


## Nginx配置高可用性怎么配置？


当上游服务器(真实访问服务器)，一旦出现故障或者是没有及时相应的话，应该直接轮训到下一台服务器，保证服务器的高可用

Nginx配置代码：

```
server {
    listen 80;
    server_name www.lijie.com;cc  nginx发送给上游服务器(真实访问的服务器)超时时间
        proxy_send_timeout 1s;###
        nginx接受上游服务器(真实访问的服务器)超时时间
        proxy_read_timeout 1s;
        index index.html index.htm;
    }
}
```


## 502错误可能原因 


**1、** FastCGI进程是否已经启动

**2、** FastCGI worker进程数是否不够

**3、** FastCGI执行时间过长

**1、** fastcgi_connect_timeout 300;

**2、** fastcgi_send_timeout 300;

**3、** fastcgi_read_timeout 300;

**FastCGI Buffer不够**

**1、** nginx和apache一样，有前端缓冲限制，可以调整缓冲参数

**2、** fastcgi_buffer_size 32k;

**3、** fastcgi_buffers 8 32k;

**Proxy Buffer不够**

**1、** 如果你用了Proxying，调整

**2、** proxy_buffer_size 16k;

**3、** proxy_buffers 4 16k;

**php脚本执行时间过长**

将php-fpm.conf的0s的0s改成一个时间


## 在 Nginx 中，解释如何在 URL 中保留双斜线?


要在 URL 中保留双斜线，就必须使用 merge_slashes_off;

语法:merge_slashes [on/off]

默认值: merge_slashes on

环境: http，server


## Nginx服务器上的Master和Worker进程分别是什么?


Master进程：读取及评估配置和维持 ；Worker进程：处理请求。


## Nginx的优缺点？


**优点：**

**1、**  占内存小，可实现高并发连接，处理响应快

**2、**  可实现http服务器、虚拟主机、方向代理、负载均衡

**3、**  Nginx配置简单

**4、**  可以不暴露正式的服务器IP地址

**缺点：**

动态处理差，nginx处理静态文件好,耗费内存少，但是处理动态页面则很鸡肋，现在一般前端用nginx作为反向代理抗住压力，

## 漏桶流算法和令牌桶算法知道，漏桶算法


漏桶算法是网络世界中流量整形或速率限制时经常使用的一种算法，它的主要目的是控制数据注入到网络的速率，平滑网络上的突发流量。漏桶算法提供了一种机制，通过它，突发流量可以被整形以便为网络提供一个稳定的流量。也就是我们刚才所讲的情况。漏桶算法提供的机制实际上就是刚才的案例：**突发流量会进入到一个漏桶，漏桶会按照我们定义的速率依次处理请求，如果水流过大也就是突发流量过大就会直接溢出，则多余的请求会被拒绝。所以漏桶算法能控制数据的传输速率。**


## 限制并发连接数 


Nginx中的ngx_http_limit_conn_module模块提供了限制并发连接数的功能，可以使用limit_conn_zone指令以及limit_conn执行进行配置。接下来我们可以通过一个简单的例子来看下：

```
 http {

     limit_conn_zone $binary_remote_addr zone=myip:10m;
     limit_conn_zone $server_name zone=myServerName:10m;
 }

 server {

     location / {

         limit_conn myip 10;
         limit_conn myServerName 100;
         rewrite / http://www.lijie.net permanent;
     }
 }
```

上面配置了单个IP同时并发连接数最多只能10个连接，并且设置了整个虚拟服务器同时最大并发数最多只能100个链接。当然，只有当请求的header被服务器处理后，虚拟服务器的连接数才会计数。刚才有提到过Nginx是基于漏桶算法原理实现的，实际上限流一般都是基于漏桶算法和令牌桶算法实现的。接下来我们来看看两个算法的介绍：


## fastcgi 与 cgi 的区别？

**cgi**

**1、** web 服务器会根据请求的内容，然后会 fork 一个新进程来运行外部 c 程序（或 perl 脚本…）， 这个进程会把处理完的数据返回给 web 服务器，最后 web 服务器把内容发送给用户，刚才 fork 的进程也随之退出。

**2、** 如果下次用户还请求改动态脚本，那么 web 服务器又再次 fork 一个新进程，周而复始的进行。

**fastcgi**

web 服务器收到一个请求时，他不会重新 fork 一个进程（因为这个进程在 web 服务器启动时就开启了，而且不会退出），web 服务器直接把内容传递给这个进程（进程间通信，但 fastcgi 使用了别的方式，tcp 方式通信），这个进程收到请求后进行处理，把结果返回给 web 服务器，最后自己接着等待下一个请求的到来，而不是退出。

综上，差别在于是否重复 fork 进程，处理请求。


## 请解释一下什么是 Nginx?


Nginx 是一个 web 服务器和反向代理服务器，用于 HTTP、HTTPS、SMTP、POP3和 IMAP 协议。


## Nginx目录结构有哪些？  


```
[root@localhost ~]# tree /usr/local/nginx
/usr/local/nginx
├── client_body_temp
├── conf                             # Nginx所有配置文件的目录
│   ├── fastcgi.conf                 # fastcgi相关参数的配置文件
│   ├── fastcgi.conf.default         # fastcgi.conf的原始备份文件
│   ├── fastcgi_params               # fastcgi的参数文件
│   ├── fastcgi_params.default       
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types                   # 媒体类型
│   ├── mime.types.default
│   ├── nginx.conf                   # Nginx主配置文件
│   ├── nginx.conf.default
│   ├── scgi_params                  # scgi相关参数文件
│   ├── scgi_params.default  
│   ├── uwsgi_params                 # uwsgi相关参数文件
│   ├── uwsgi_params.default
│   └── win-utf
├── fastcgi_temp                     # fastcgi临时数据目录
├── html                             # Nginx默认站点目录
│   ├── 50x.html                     # 错误页面优雅替代显示文件，例如当出现502错误时会调用此页面
│   └── index.html                   # 默认的首页文件
├── logs                             # Nginx日志目录
│   ├── access.log                   # 访问日志文件
│   ├── error.log                    # 错误日志文件
│   └── nginx.pid                    # pid文件，Nginx进程启动后，会把所有进程的ID号写到此文件
├── proxy_temp                       # 临时目录
├── sbin                             # Nginx命令目录
│   └── nginx                        # Nginx的启动命令
├── scgi_temp                        # 临时目录
└── uwsgi_temp                       # 临时目录
```


## 请解释你如何通过不同于 80 的端口开启 Nginx?


为了通过一个不同的端口开启 Nginx，你必须进入/etc/Nginx/sites

enabled/，如果这是默认文件，那么你必须打开名为“default”的文件。编辑

文件，并放置在你想要的端口：

Like server { listen 81; }


## 解释如何在 Nginx 中获得当前的时间?


要获得 Nginx 的当前时间，必须使用 SSI 模块、$$date_gmt 和$$date_local 的变

量。

Proxy_set_header THE-TIME $date_gmt;


## 使用“反向代理服务器的优点是什么?


反向代理服务器可以隐藏源服务器的存在和特征。它充当互联网云和web服务器之间的中间层。这对于安全方面来说是很好的，特别是当您使用web托管服务时。


## 请解释 Nginx 如何处理 HTTP 请求？


**1、** 首先，Nginx 在启动时，会解析配置文件，得到需要监听的端口与 IP 地址，然后在 Nginx 的 Master 进程里面先初始化好这个监控的Socket(创建 S ocket，设置 addr、reuse 等选项，绑定到指定的 ip 地址端口，再 listen 监听)。

**2、** 然后，再 fork(一个现有进程可以调用 fork 函数创建一个新进程。由 fork 创建的新进程被称为子进程 )出多个子进程出来。

**3、** 之后，子进程会竞争 accept 新的连接。此时，客户端就可以向 nginx 发起连接了。当客户端与nginx进行三次握手，与 nginx 建立好一个连接后。此时，某一个子进程会 accept 成功，得到这个建立好的连接的 Socket ，然后创建 nginx 对连接的封装，即 ngx_connection_t 结构体。

**4、** 接着，设置读写事件处理函数，并添加读写事件来与客户端进行数据的交换。


## fair(第三方插件)


必须安装upstream_fair模块。

对比 weight、ip_hash更加智能的负载均衡算法，fair算法可以根据页面大小和加载时间长短智能地进行负载均衡，响应时间短的优先分配。

```
upstream backserver {
 server server1; 
 server server2; 
 fair; 
}
```

哪个服务器的响应速度快，就将请求分配到那个服务器上。

## Nginx负载均衡方式

六种负载均衡方式：（作用也是避免请求过了集中到一台机器）

> 1. 轮询
> 2. weight权重方式
> 3. ip_hash根据ip分配方式，可以让同一个用户访问同一个资源。
> 4. least_conn最少链接方式
> 5. fair响应时间方式，请求向响应时间较短的机器发。
> 6. url_hash依据URL分配方式，根据url请求到同一个服务器，避免缓存资源多次请求。

## Nginx配置相关

1. event标签里面，有**worker_processer**（表示连接处理请求的线程个数，一般和cpu核数保持一致），**worker_connections**（每一个worker最大连接数，一般和ulimit -a中的openfile参数[这个参数表示操作系统最大打开文件数量，最好改大些，我见过netty实战中直接把它改成100万了]除以worker_processer数）。
2. **keepalive_timeout、send timeout **请求和响应的超时时间。
3. **default_type  application/octet-stream;**将一些路径下的例如文件通知到浏览器是文件流。
4. **client_max_body_size 10m;**上传文件大小限制。
5. 转发配置


	server{
		  listen 80;
		  charset utf-8;
		  server_name www.ican.com;
		  access_log logs/host.access.log;
		  location /{
			  proxy_pass http://127.0.0.1:7001;
		  }
	}

以下知识来自：https://www.cnblogs.com/binghe001/p/13273114.html

## Nginx限流模块

1、limit_req_zone：限制单位时间内的请求数，漏桶算法。

2、limit_req_conn：限制同一时间连接数，即并发限制。

**limit_req_zone参数说明**

```bash
Syntax: limit_req zone=name [burst=number] [nodelay];
Default:    —
Context:    http, server, location
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
```

- 第一个参数：$binary_remote_addr 表示通过remote_addr这个标识来做限制，“binary_”的目的是缩写内存占用量，是限制同一客户端ip地址。
- 第二个参数：zone=one:10m表示生成一个大小为10M，名字为one的内存区域，用来存储访问的频次信息。
- 第三个参数：rate=1r/s表示允许相同标识的客户端的访问频次，这里限制的是每秒1次，还可以有比如30r/m的。

```bash
limit_req zone=one burst=5 nodelay;
```

- 第一个参数：zone=one 设置使用哪个配置区域来做限制，与上面limit_req_zone 里的name对应。
- 第二个参数：burst=5，重点说明一下这个配置，burst爆发的意思，这个配置的意思是设置一个大小为5的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。
- 第三个参数：nodelay，如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回503，如果没有设置，则所有请求会等待排队。

示例：

**limit_req_zone示例**

```bash
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    server {
        location /search/ {
            limit_req zone=one burst=5 nodelay;
        }
}
```

## ngx_http_limit_conn_module 参数说明

这个模块用来限制单个IP的请求数。并非所有的连接都被计数。只有在服务器处理了请求并且已经读取了整个请求头时，连接才被计数。

```bash
Syntax: limit_conn zone number;
Default:    —
Context:    http, server, location
limit_conn_zone $binary_remote_addr zone=addr:10m;
 
server {
    location /download/ {
        limit_conn addr 1;
    }
```

一次只允许每个IP地址一个连接。

```bash
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;
 
server {
    ...
    limit_conn perip 10;
    limit_conn perserver 100;
}
```

可以配置多个limit_conn指令。例如，以上配置将限制每个客户端IP连接到服务器的数量，同时限制连接到虚拟服务器的总数。

##  Nginx 反向代理负载均衡

普通的负载均衡软件,如 LVS,其实现的功能只是对请求数据包的转发、传递,从负载均衡下的节点服务器来看,接收到的请求还是来自访问负载均衡器的客户端的真实用户;而反向代理就不一样了,反向代理服务器在接收访问用户请求后,会代理用户 重新发起请求代理下的节点服务器,最后把数据返回给客户端用户。在节点服务器看来,访问的节点服务器的客户端用户就是反向代理服务器,而非真实的网站访问用户。

## upstream_module 和健康检测

ngx_http_upstream_module 是负载均衡模块,可以实现网站的负载均衡功能即节点的健康检查,upstream 模块允许 Nginx 定义一组或多组节点服务器组,使用时可通过 proxy_pass 代理方式把网站的请求发送到事先定义好的对应 Upstream 组 的名字上。

| upstream 模块内参数 | 参数说明                                                     |
| ------------------- | ------------------------------------------------------------ |
| weight              | 服务器权重                                                   |
| max_fails           | Nginx 尝试连接后端主机失败的此时,这是值是配合 proxy_next_upstream、fastcgi_next_upstream 和 memcached_next_upstream 这三个参数来使用的。当 Nginx接收后端服务器返回这三个参数定义的状态码时,会将这个请求转发给正常工作的的后端服务器。如 404、503、503,max_files=1 |
| fail_timeout        | max_fails 和 fail_timeout 一般会关联使用,如果某台 server 在 fail_timeout 时间内出现了max_fails 次连接失败,那么 Nginx 会认为其已经挂掉,从而在 fail_timeout 时间内不再去请求它,fail_timeout 默认是 10s,max_fails 默认是 1,即默认情况只要是发生错误就认为服务器挂了,如果将 max_fails 设置为 0,则表示取消这项检查 |
| backup              | 表示当前 server 是备用服务器,只有其它非 backup 后端服务器都挂掉了或很忙才会分配请求给它 |
| down                | 标志服务器永远不可用,可配合 ip_hash 使用                     |

```nginx
upstream lvsServer{
server 191.168.1.11 weight=5 ;
server 191.168.1.22:82;
server example.com:8080 max_fails=2 fail_timeout=10s backup;
#域名的话需要解析的哦,内网记得 hosts
}
```

## proxy_pass 请求转发

proxy_pass 指令属于 ngx_http_proxy_module 模块,此模块可以将请求转发到另一台服务器,在实际的反向代理工作中,会通过 location 功能匹配指定的 URI,然后把接收到服务匹配 URI 的请求通过 proyx_pass 抛给定义好的 upstream 节点池。

```nginx
location /download/ {
proxy_pass http://download/vedio/;
}
#这是前端代理节点的设置
#交给后端 upstream 为 download 的节点
```

| proxy 模块参数             | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| proxy_next_upstream        | 什么情况下将请求传递到下一个 upstream                        |
| proxy_limite_rate          | 限制从后端服务器读取响应的速率                               |
| proyx_set_header           | 设置 http 请求 header 传给后端服务器节点,如:可实现让代理后端的服务器节点获取访问客户端的这是 ip |
| client_body_buffer_size    | 客户端请求主体缓冲区大小                                     |
| proxy_connect_timeout      | 代理与后端节点服务器连接的超时时间                           |
| proxy_send_timeout         | 后端节点数据回传的超时时间                                   |
| proxy_read_timeout         | 设置 Nginx 从代理的后端服务器获取信息的时间,表示连接成功建立后,Nginx 等待后端服务器的响应时间 |
| proxy_buffer_size          | 设置缓冲区大小                                               |
| proxy_buffers              | 设置缓冲区的数量和大小                                       |
| proyx_busy_buffers_size    | 用于设置系统很忙时可以使用的 proxy_buffers 大小,推荐为proxy_buffers*2 |
| proxy_temp_file_write_size | 指定 proxy 缓存临时文件的大小                                |

