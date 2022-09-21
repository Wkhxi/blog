---
title: nginx.conf配置详解
tags:
  - Nginx
  - Linux
categories:
  - Nginx
  - 3. nginx.conf 配置
abbrlink: f055b838
date: 2022-08-10 23:21:54
---



```
配置文件的路径（相对于nginx文件夹）： nginx/conf/nginx.config
```

------



```shell
# 开启的 worker子进程数 1个，这个个数一般等于服务器cpu的内核数

worker_processes  1;
```



```shell
# 每个worker进程 最大连接数

events {
    worker_connections  1024;
}
```


http块

```shell
http {

    #引入 mime.types 文件
    # mime.types 文件中 写了 http头中返回的文件类型 与 后缀名 的映射关系
    # 作用：用来告诉 浏览器解析的是什么类型的文件
    include       mime.types;
    # 默认类型
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;


    # 一个 server 代表一个 主机
    # 一个nginx可以运行配置多个主机
    # 每个主机 都有自己独立的 根目录 互不干扰
    # 即 虚拟主机
    server {
    	  # 监听的端口号
        listen       80;
        # 主机名
        server_name  localhost;
	
      # 用于匹配uri
      # 域名后面的 路径部分
      # 比如：http://testabc.com/xxx/index.html 中的 /xxx/index.html
        location / {
            # html是相对路径 nginx文件夹下的html文件夹
            # 当匹配上uri后，从哪个目录下找
            root   html;
            index  index.html index.htm;
        }

		
        error_page   500 502 503 504  /50x.html;
        # /50x.html 的错误页 去 html下找
        location = /50x.html {
            root   html;
        }

    }

}
```

------





#### sendfile 零拷贝

------

```
当 没有开启 sendfile：

当nginx接收到网络请求的时候（有一次拷贝到应用程序的内存的过程）：
	1. nginx应用程序需要将所需文件从硬盘中读取出来 到 应用程序的内存中
	2. 将 应用程序的内存中 的文件发送给计算机的 网络接口
	3. 网络接口 将文件推送给用户

```


```
当 开启 sendfile：

当nginx接收到网络请求的时候：
	1. nginx应用程序 给 网络接口推送信号 sendfile，告诉网络接口需要读取哪些文件
	2. 网络接口 读取文件
	3. 网络接口 将文件推送给用户

```





------