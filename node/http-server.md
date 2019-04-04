# http-server: 一个命令行里的 http 服务器
> [原文地址](https://www.npmjs.com/package/http-server)
> ```version 0.11.1```

http-server 是一个简单的，零配置的命令行里的 http 服务器。
它足够强大到在生产环境中使用。
但是它又是足够简单且可开发（hackable），以用来做测试、本地开发、学习。

![龟图](https://images.gitee.com/uploads/images/2019/0122/113001_51108d4c_1517186.png "public.png")

### 全局安装
通过 npm 安装
```
npm install http-server -g
```
这样就可以全局安装 http-server，并且可以在命令行里启动它。
##### 用法
```
http-server [path] [options]
```
[path] 的默认值是 ```./public```，如果这个文件夹存在的话。否则就是 ```./```。

*现在你可以访问 ```http://localhost:8080``` 来看看你的服务器了。*
> 译者注：如果你的机器上有其他服务器，那么端口可能会变成其他的数值，具体的数值可以看启动后控制台输出的日志

##### 可选的配置
> 译者注：这里指 [options] 参数

+ -p 端口号（默认值是 8080）
+ -a 地址 （默认值是 0.0.0.0）
+ -d 显示目录监听 （默认值是 true）
+ -i 展示自动索引（autoIndex）（默认值是 true）
+ -g 或 --gzip 当设置为 true（默认值是 false）时，它会优先使用 gzipped 版本的文件（如果不存在，则使用非 gzipped 版本）；并且接受 gzip 编码的请求
+ -e 或 --ext 默认的文件扩展名如果没提供的话（默认值 html）
+ -s 或 --silent 控制台不输出日志信息
+ --cors 通过头部 Access_Control-Allow-Origin 设置 CORS 
+ -o 在服务器开启之后，启动浏览器窗口
+ -c 为头部 cache-control max-age 设置缓存时间（单位：秒），比如 -c10 代表 10 秒（默认值 3600）。可以使用 -c-1 来关闭缓存
+ -U 或 --utc 在日志信息里使用 UTC 格式的时间
+ -P 或 --proxy 代理所有本地不能解析的请求到给定的 url，比如：-P http://someurl.com
+ -S 或 --ssl 开启 https
+ -C 或 --cert ssl 证书文件的路径（默认值 cert.pem)
+ -K 或 --key ssl 密钥文件的路径（默认值 key.pem）
+ -r 或 --robots 提供一个 /robots.txt（它的内容默认为 'User-agent:*\nDisallow:/'）
+ -h 或 --help 打印这个列表并且退出

### 开发
checkout [这个仓库](https://github.com/indexzero/http-server#readme)到本地，然后
```
$ npm i
$ node bin/http-server
```
现在你可以访问 http://localhost:8080 来看你的服务器

你应该可以在这个 URL 看到上面那个龟图。
demo 内容在 ./public 文件夹下。


