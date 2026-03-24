---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/PHP principle/nginx+php(工作原理)/","title":"nginx+php(工作原理)","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:47:18.000+08:00","updated":"2026-03-24T17:46:40.471+08:00"}
---

php工作原理

首先先了解下常听说的cgi，php-cgi，fastcgi，php-fpm到底是什么关系，帮助了解php的工作原理



cgi协议

cgi协议用来确定webserver（例如nginx），也就是内容分发服务器传递过来什么数据，什么样格式的数据



php-cgi进程解释器

php-cgi是php的cgi协议进程解释器，每次启动时，需要经历加载php.ini文件->初始化执行环境->处理请求->返回内容给webserver->php-cgi进程退出的流程



fastcgi协议

fastcgi协议是对cgi协议效率提升的补充，主要是针对每次请求过来时都需要启动一个cgi解释器进程的优化，不再需要cgi解释器进程每次收到webserver请求后都需要重新加载php.ini文件和初始化执行环境



php-fpm进程管理器

php-fpm是对fastcgi协议的实现，是进程管理器，启动时包括master和worker进程俩部分，master进程负责管理worker进程，worker进程一般具有多个，用来监听端口，接收来自webserver请求，且每个worker进程都有一个cgi进程解释器，用来执行php代码



php启动和工作原理

启动phpfpm时，会启动master进程，加载php.ini文件，初始化执行环境，并启动多个worker进程。每次请求来时监听端口的worker进程会进行处理



php平滑重启原理

每次修改完php.ini配置并重启后，会启动新的worker进程加载新的配置，而之前已经存在的进程会在工作完成之后销毁，因此实现平滑重启



nginx工作原理

如果想弄明白nginx和php配合的原理，还需要先了解nginx的配置文件中的server部分

```shell
server {
    # 监听80端口，接收http请求
    listen       80;
    # 一般存放网址，表示配置的哪个项目
    server_name  www.example.com;
    # 存放代码的根目录地址或代码启动入口
    root /home/wwwroot/zensmall/public/;
    # 网站默认首页
    index index.php index.html;
    
    # 当请求网站的url进行location的前缀匹配且最长匹配字符串是该配置项时，
    # 按顺序检查文件是否存在，并返回第一个找到的文件
    location / {
          # try_files，按顺序检查文件是否存在，返回第一个找到的文件
          # $uri代表不带请求参数的当前地址
          # $query_string代表请求携带的参数
          # 按顺序检查$uri文件，$uri地址是否存在，如果存在，返回第一个找到的文件；
          # 如果都不存在，发起访问/index.php?$query_string的内部请求，该请求会重新匹配到下面的location请求
          try_files   $uri $uri/ /index.php?$query_string; 
    }
    # 当请求网站的php文件的时候，反向代理到php-fpm去处理
    location ~ \.php$ {
          # 引入fastcgi的配置文件
          include       fastcgi_params; 
          # 设置php fastcgi进程监听的IP地址和端口
          fastcgi_pass   127.0.0.1:9000;
          # 设置首页文件
          fastcgi_index  index.php;
          # 设置脚本文件请求的路径
          fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    }
}
```

上面server配置的整体含义是：每次nginx监听到80端口的url请求，会对url进行location匹配。如果匹配到/规则时，会进行内部请求重定向，发起/index.php?$query_string的内部请求，而对应的location配置规则会将请求发送给监听9000端口的php-fpm的worker进程



总结

下面总结下最简单的用户请求流程：

用户访问域名->域名DNS解析->请求到IP服务器和端口->nginx监听到端口的请求->nginx对url进行location匹配->执行匹配location下的规则->nginx转发请求给php->php-fpm的worker进程监听到nginx请求->worker进程执行请求->worker进程返回执行结果给nginx->nginx返回结果给用户