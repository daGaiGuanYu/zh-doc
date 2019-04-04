> 本篇翻译[原文地址](http://nginx.org/en/docs/beginners_guide.html#control)
# 初学者指导

+ 启动、停止和重载配置
+ 配置文件的结构
+ 提供静态文件下载服务
+ 搭建一个简单的代理服务器
+ 搭建 FastCGI 代理  

这篇指导简单介绍 nginx 并且描述 nginx 可以做的一些简单的任务。现在假定 nginx 已经被安装到读者的机器上，如果还没安装的话，请参考[安装页](http://nginx.org/en/docs/install.html)。这篇指导描述如何启动和停止 nginx、重载它的配置、解释配置文件结构、描述如何设置 nginx 以用来作为静态文件服务器、如何配置 nginx 来使它成为一个代理服务器、如何将它与一个 FastCGI 应用连接。

nginx 有一个主进程和多个工作进程（worker process）。主进程的主要目的是读取和评估配置，并且维护工作进程。实际上，是工作进程来处理请求。nginx 使用以事件为基础的模型和依赖操作系统的机制（OS-dependent mechanisms）来在工作进程之间高效地分发请求。工作进程的数量定义在配置文件中，数量可以是一个固定值（在配置文件中给定）或者根据可用的 CPU 核心的数量自动调整（详参[工作进程](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)）（译者注：也就是说，你可以在配置文件中设置一个固定值，也可以不设置，不设置的话，nginx 就会根据 CPU 的核心的数量计算一个合理的数值）。

nginx 和它的模块的工作方式取决于配置文件。默认的，配置文件的文件名是 nginx.conf，被放置在 /usr/local/nginx/conf、/etc/nginx、或者 /usr/local/etc/nginx 目录下。

## 启动、停止和重载配置
运行可执行文件（the executable）以启动 nginx。一旦 nginx 被启动，可以通过调用那个可执行文件并附带 -s 参数来控制它。使用下面的语法：
```
nginx -s signal
```
signal 可以是下列中的一个
+ stop — 快速停止
+ quit — 优雅的（graceful）停止
+ reload — 重载配置
+ reopen — 重新打开日志文件
    
例如，停止 nginx 并且等待工作进程完成处理当前的请求，执行下面的命令：
```
nginx -s quit
```
这个命令应该被启动 nginx 的用户执行。

配置文件的改动不会立即被应用，你可以向 nginx 发送重载命令或者重启。请执行下面命令以重载配置文件：
```
nginx -s reload
```

一旦主进程收到重载配置的信号，它会检查新配置的语法的有效性，并且尝试应用新的配置。如果成功了，主进程就会开启新的工作进程，并且发送信号给老的工作进程，请求它们关闭。否则（译者注：新配置语法存在错误，或应用不成功），主进程回滚改变，并且继续和老的工作进程工作。老工作进程收到停止的命令后，停止接受新的连接并且继续处理当前请求，直到所有请求被处理。在这之后，老工作进程退出。


信号也可以通过 unix 工具（例如 kill 工具）被发送给 nginx。这种情况下，信号被直接发送给指定进程 id 的进程。nginx 主进程的进程号被写入到 /usr/local/nginx/logs 或者 /var/run 下的 nginx.pid 文件（默认情况下）。比如，如果主进程的进程 id 是 1628，那么想通过发信号的方式让 nginx 优雅地（graceful）停止，可以执行：
```
kill -s QUIT 1628
```
为了得到正在运行的 nginx 进程的列表（list），可以使用 ps 工具。比如，通过以下方式：
```
ps -ax | grep nginx
```
想知道更多关于发送信号给 nginx 的信息，参阅[控制 nginx](http://nginx.org/en/docs/control.html)。

## 配置文件的结构
配置文件里声明了各种指令，用来控制 nginx 的各个模块。指令分为两种：简单指令和块指令。简单指令包含命令名和参数，并用空格隔开，以分号结束。块指令的结构和简单指令一样，但是块指令的结尾不是分号，而是一个用括号包围起来的附加指令（additional instruction）的集合。如果一个块指令的括号里可以有其他指令，那么这种块指令叫做上下文（context）（例如：events, http, server, 和 location）。

配置文件中，在任何上下文之外的指令被认为处于主上下文中（main context）。events 和 http 指令被放在主上下文中，server 被放在 http 上下文中，location 被放在 server 上下文中。

配置文件中的注释，写在 # 之后。

## 提供静态文件下载服务
web 服务器的一个重要任务就是提供静态文件下载（比如图片和静态 HTML 页面）。你可以实现这样一个例子：提供在不同目录下的静态文件的下载服务（实际情况也取决于需求）：/data/www（可能用来放 HTML 文件）和 /data/image（放图片）。这需要编辑配置文件，在 http 块中设置一个 server 块，并且包含两个 location 块：
```
http {
  server {
  }
}
```

通常情况下，配置文件包含若干 server 块（区别在于监听的端口，和 server 的名字）。一旦 nginx 决定哪个 server 进程处理一个请求，它就会比较声明在请求头里的 URI 和 server 块中 location 指令的参数。


添加下列 location 块到 server 块中：
```
location / {
  root /data/www;
}
```
这个 location 块声明 “/” 前缀，和请求中 URI 比较。对于匹配到的请求，URI 将被添加到 root 指令中声明的路径之后，也就是 /data/www，以此来组成所请求文件在本地文件系统中的路径。如果有多个匹配成功的 location 块，nginx 将会选择最长前缀的那一个。上面的 location 块提供了一个最短的前缀，长度为一。只有当其他 location 块都匹配失败，这个块才会被使用。

接下来，添加第二个 location 块：
```
location /images/ {
  root /data;
}
```
以 /images/ 开头的请求会匹配成功（ “/” 也会匹配成功，但是前缀较短）。

现在配置文件中的 server 块，应该是下面这样子：
```
server {
  location / {
    root /data/www;
  }
  location /images/ {
    root /data;
  }
}
```

这已经是一个监听在 80 端口的服务器的有效配置，并且可以在本地通过 http://localhost/ 访问到。作为以 /images/ 为开头的 URI 的请求的响应，服务器将会从 /data/images/ 目录下发送文件。例如，对于这个请求：http://localhost/images/example.png，nginx 将会响应 /data/images/example.png。如果这个文件不存在，nginx 将会响应 404 错误。URI 不以 /images/ 开头的请求将被映射到 /data/www/ 目录。例如，请求 http://localhost/some/example.html，nginx 将响应 /data/www/some/example.html 文件。

为了应用新配置，如果 nginx 没被启动的话，就启动 nginx，否则向 nginx 主进程发送重载命令：
```
nginx -s reload
```
> 也许某些东西不按预期运行，你可以尝试在 access.log 和 error.log 里找到原因（这两个文件在 /usr/local/nginx/logs 或者 /var/log/nginx 目录下）。


## 搭建一个简单的代理服务器
nginx 的一个被频繁使用的功能就是搭建一个代理服务器（接收请求，并把这个请求传给被代理的服务器，然后收到被代理服务器的响应，最后返回给客户端）。

我们将配置一个基础的代理服务器，它提供对图片文件（译者注：此处存疑，原文是“images with files”）的下载并且把其他请求发送给被代理服务器。这个例子中，两个服务器被定义在一个单独的 nginx 实例中。

首先，再添加一个 server 块到配置文件，以定义被代理的服务器：
```
server {
  listen 8080;
  root /data/up1;

  location / {
  }
}
```

这是一个简单的服务器，监听在 8080 端口（之前，监听指令未被声明，nginx 会默认监听 80 端口）。它将映射所有 /data/up1 请求到本地文件系统。创建这个目录，并且把 index.html 放进去。注意，root 指令放到 server 上下文中。当被选中作为处理请求的 location 块没有自己的 root 指令时，使用这样的 root 指令。

接下来，使用前面部分的 server 配置，做出一点修改，使它成为代理服务器。在第一个 location 块中，添加 proxy_pass 指令，并附带生被代理服务器的协议、名称（在我们的例子中，是 http://localhost:8080）：
```
server {
  location / {
    proxy_pass http://localhost:8080;
  }
  location /images/ {
    root /data;
  }
}
```

然后我们修改第二个 location 块，它当前映射所有的以 /images/ 为前缀的请求到 /data/images 目录下的文件。修改之后，使它匹配特定扩展名的图片请求。修改后的 location 块如下：
```
location ~ \.(gif|jpg|png)$ {
  root /data/images;
}
```
这个参数是一个正则表达式，匹配所有以 .gif、.jpg 和 .png 结尾的 URI。正则表达式必须在前面放一个符号：~。对应的请求会被映射到 /data/images 目录下。

当 nginx 选择哪个 location 块来处理一个请求时，首先检查声明了前缀了的 location 指令，记住带有最长前缀的 location，之后检查正则表达式。如果用正则表达式匹配到，nginx 就会选择这个 location，否则选择之前记住的那个。

现在的代理服务器的配置是这样的：
```
server {
  location / {
    proxy_pass http://localhost:8080/;
  }
  location ~ \.(gif|jpg|png)$ {
    root /data/images;
  }
}
```

这个服务器会过滤所有以 gif、jpg 或者 png 为结尾的请求，并且把他们映射到 /data/images 目录下（通过添加 URI 到 root 指令的参数后面）并且把所有其他的请求传递给之前配置的被代理的服务器。

为了应用新配置，发送重载信号给 nginx（像之前部分描述的那样）。

还有[更多](nginx.org/en/docs/http/ngx_http_proxy_module.html)配置代理连接的指令。

## 搭建一个代理 FastCGI
nginx 可以用来 route 一个到 FastCGI 服务器的请求，不管这个服务器上运行的程序是哪个框架、何种语言（例如 PHP）。最基础的使 nginx 和一个 FastCGI 服务器工作的配置包含使用 fastcgi_pass 指令，而不是 proxy_pass 指令。fastcgi_param 指令可以用来设置传给 FastCGI 服务器的参数。假定 FastCGI 服务器可以在 localhost:9000 访问到。用上面的代理部分的配置作为基础，用 fastcgi_pass 代替 proxy_pass 指令，然后改变它的参数为 localhost:9000。在 PHP 中，SCRIPT_FILENANME 参数被用来决定脚本名称，QUERY_STRING 参数被用来传递请求参数。当前的配置是这样的：
```
server {
  location / {
    fastcgi_pass  localhost:9000;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param QUERY_STRING    $query_string;
  }
  location ~ \.(gif|jpg|png)$ {
    root /data/images;
  }
}
```
这样就搭建了一个服务器，它可以通过 FastCGI 协议 route 所有的请求（除了静态图片）到运行在 localhost:9000 的被代理的服务器。