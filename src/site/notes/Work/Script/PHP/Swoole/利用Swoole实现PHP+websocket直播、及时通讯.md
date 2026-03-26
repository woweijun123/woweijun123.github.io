---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Swoole/利用Swoole实现PHP+websocket直播、及时通讯/","title":"利用Swoole实现PHP+websocket直播、及时通讯","tags":["flashcards"],"noteIcon":"","created":"2026-03-10T22:33:54.000+08:00","updated":"2026-03-24T17:49:56.533+08:00","dg-note-properties":{"title":"利用Swoole实现PHP+websocket直播、及时通讯","tags":["flashcards"],"reference linking":null}}
---

websocket
Websocket只是一个网络通信协议，就像 http、ftp等都是网络通信的协议一样；
相对于HTTP这种非持久的协议来说，Websocket是一个持久化网络通信的协议；
WebSocket和HTTP的关系
![](https://weichengjun2.dpdns.org/i/2023/09/21/650bea3335df2.png)

有交集，但是并不是全部。
Websocket只是借用了HTTP的一部分协议来完成一次握手。(HTTP的三次握手，此处只完成一次)
Http 协议
![](https://weichengjun2.dpdns.org/i/2023/09/21/650bea426fe68.png)

第一次握手：主机A发送位码为syn＝1,随机产生seq number=1234567的数据包到服务器，主机B由SYN=1知道，A要求建立联机；

第二次握手：主机B收到请求后要确认联机信息，向A发送ack number=(主机A的seq+1),syn=1,ack=1,随机产生seq=7654321的包

第三次握手：主机A收到后检查ack number是否正确，即第一次发送的seq number+1,以及位码ack是否为1，若正确，主机A会再发送ack number=(主机B的seq+1),ack=1，主机B收到后确认seq值与ack=1则连接建立成功。

完成三次握手，主机A与主机B开始传送数据。



Long poll 和Ajax轮询以及WebSocket的原理

1、Ajax轮询的原理。

场景如下：
客户端：啦啦啦，有没有新信息(Request)
服务端：没有（Response）
客户端：啦啦啦，有没有新信息(Request)
服务端：没有。。（Response）
客户端：啦啦啦，有没有新信息(Request)
服务端：你好烦啊，没有啊。。（Response）
客户端：啦啦啦，有没有新消息（Request）
服务端：好啦好啦，有啦给你。（Response）
客户端：啦啦啦，有没有新消息（Request）
服务端：。。。。。没。。。。没。。。没有（Response） ---- loop
代码：
```javascript
var polling = function (url, type, data) {
    var xhr = new XMLHttpRequest(),
        type = type || "GET",
        data = data || null;

    xhr.onreadystatechange = function () {
        if (xhr.readyState == 4) {
            receive(xhr.responseText);
            xhr.onreadystatechange = null;
        }
    };

    xhr.open(type, url, true);
    //IE的ActiveXObject("Microsoft.XMLHTTP")支持GET方法发送数据，
    //其它浏览器不支持，已测试验证
    xhr.send(type == "GET" ? null : data);
};

var timer = setInterval(function () {
    polling();
}, 1000);
```



2、long poll的原理。

long poll 其实原理跟 ajax轮询 差不多，都是采用轮询的方式，不过采取的是阻塞模型（一直打电话，没收到就不挂电话），也就是说，

客户端发起连接后，如果没消息，就一直不返回Response给客户端。直到有消息才返回，返回完之后，客户端再次建立连接，周而复始。

场景如下：

客户端：啦啦啦，有没有新信息，没有的话就等有了才返回给我吧（Request）

服务端：额。。 (等待到有消息的时候)。。来 给你（Response）

客户端：啦啦啦，有没有新信息，没有的话就等有了才返回给我吧（Request）

客户端：啦啦啦啦，有新信息么？

服务端：月线正忙，请稍后再试（503 Server Unavailable）

客户端：。。。。好吧，啦啦啦，有新信息么？

服务端：月线正忙，请稍后再试（503 Server Unavailable）

客户端：

![](https://weichengjun2.dpdns.org/i/2023/09/21/650bea44dfe47.png)

代码：

```javascript
var longPoll = function (type, url) {
    var xhr = new XMLHttpRequest();

    xhr.onreadystatechange = function () {
        // 状态为 4，数据传输完毕，重新连接
        if (xhr.readyState == 4) {
            receive(xhr.responseText);
            xhr.onreadystatechange = null;

            longPoll(type, url);
        }
    };

    xhr.open(type, url, true);
    xhr.send();
}
```



然后服务端在一旁忙的要死：冰箱，我要更多的冰箱！更多。。更多。。

从上面可以看出其实这两种方式，都是在不断地建立HTTP连接，然后等待服务端处理，可以体现HTTP协议的另外一个特点，被动性。

何为被动性呢，其实就是，服务端不能主动联系客户端，只能有客户端发起。

缺陷:

从上面很容易看出来，不管怎么样，上面这两种都是非常消耗资源的。

ajax轮询 需要服务器有很快的处理速度和资源。（速度）

long poll 需要有很高的并发，也就是说同时接待客户的能力。（场地大小）

所以ajax轮询 和long poll 都有可能发生这种情况。



3、WebSocket的原理。

这时Websocket出现了。

他解决了HTTP的这几个难题。

首先，被动性，当服务器完成协议升级后（HTTP->Websocket），服务端就可以主动推送信息给客户端啦。

所以上面的情景可以做如下修改。

客户端：啦啦啦，我要建立Websocket协议，需要的服务：chat，Websocket协议版本：17（HTTP Request）

服务端：ok，确认，已升级为Websocket协议（HTTP Protocols Switched）

客户端：麻烦你有信息的时候推送给我噢。。

服务端：ok，有的时候会告诉你的。

服务端：balabalabalabala

服务端：balabalabalabala

服务端：哈哈哈哈哈啊哈哈哈哈

服务端：笑死我了哈哈哈哈哈哈哈



但是Websocket只需要一次HTTP握手，所以说整个通讯过程是建立在一次连接/状态中，也就避免了HTTP的非状态性，服务端会一直知道你的信息，直到你关闭请求，这样就解决了接线员要反复解析HTTP协议，还要查看identity info的信息。

同时由客户主动询问，转换为服务器（推送）有信息的时候就发送（当然客户端还是等主动发送信息过来的。。），没有信息的时候就交给接线员（Nginx），不需要占用本身速度就慢的客服（Handler）了

---
php安装swoole

1. 下载swoole安装

```shell
wget http://pecl.php.net/get/swoole-1.9.1.tgz
tar -zxvf swoole-1.9.1.tgz
cd swoole-1.9.1
phpize
./configure
make
make install
```



2. 在php.ini添加swoole.so

```shell
extension=swoole.so
```

![](https://weichengjun2.dpdns.org/i/2023/09/21/650bea471fa03.png)

环境依赖

- 仅支持Linux，FreeBSD，MacOS，3类操作系统

- Linux内核版本2.3.32以上

- PHP5.3.10以上版本

- gcc4.4以上版本或者clang

- cmake2.4+，编译为libswoole.so作为C/C++库时需要使用cmake 

PHP版本依赖

- swoole仅支持PHP5.3.10或更高版本，建议使用PHP5.4+

- swoole不依赖php的stream、sockets、pcntl、posix、sysvmsg等扩展。PHP只需安装最基本的扩展即可 

PHP直播代码

1.start.php 使用时需要开启，服务器输入（php start.php）

```php
<?php
//php在线直播示例代码
//使用PHPCLI模式运行
//命令：php start.php

//设置路径
define('_ROOT_', dirname(__FILE__));
require_once _ROOT_.'/function.php';
//监听地址和端口
$server = new swoole_websocket_server("0.0.0.0（这里就是四个0，不要改）", 8888);
//服务端接收连接事件
$server->on('open', function (swoole_websocket_server $server, $request) {
    if(!file_exists(_ROOT_.'/client/'.$request->fd.'.client')){
        @file_put_contents(_ROOT_.'/client/'.$request->fd.'.client',$request->fd);
    }
});
//服务端接收信息事件
$server->on('message', function (swoole_websocket_server $server, $frame) {
    foreach(notice(_ROOT_.'/client/') as $v){
            $server->push($v,$frame->data);
    }
});
//服务端接收关闭事件
$server->on('close', function ($ser, $fd) {
    @unlink(_ROOT_.'/client/'.$fd.'.client');
});
//服务开启
$server->start();
```

2.index.html 直播页面，访问该页面观看直播

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>在线直播界面</title>
</head>
<body>
<img id="receiver" style='width:640px;height:480px'/>
<script type="text/javascript" charset="utf-8">
    var ws = new WebSocket("ws://改成自己服务器ip:8888");
    var image = document.getElementById('receiver');
    ws.onopen = function(){

    }
    ws.onmessage = function(data)
    {
        image.src=data.data;
    }
</script>
</body>
</html>
```

3.rec.html主播录制页面，访问该页面进行直播录制

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>主播录制界面</title>
</head>
<body>
<video id="video" autoplay="" style='width:640px;height:480px'></video>
<canvas id="output" style="display:none"></canvas>
<script type="text/javascript" charset="utf-8">
    var ws = new WebSocket("ws://自己服务器ip:8888");
    var back = document.getElementById('output');
    var backcontext = back.getContext('2d');
    var video = document.getElementById("video");
    var success = function(stream){
        video.src = window.URL.createObjectURL(stream);
    }
    ws.onopen = function(){
        draw();
    }
    var draw = function(){
        try{
            backcontext.drawImage(video,0,0, back.width, back.height);
        }catch(e){
            if (e.name == "NS_ERROR_NOT_AVAILABLE") {
                return setTimeout(draw, 100);
            } else {
                throw e;
            }
        }
        if(video.src){
            ws.send(back.toDataURL("image/jpeg", 0.5));
        }
        setTimeout(draw, 100);
    }
    navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia ||
    navigator.mozGetUserMedia || navigator.msGetUserMedia;
    navigator.getUserMedia({video:true, audio:false}, success, console.log);
</script>
</body>
</html>
```

4.function.php 统计数据页面

```php
<?php
//统计在线人数
function clearDir($dir)
{
    $n = 0;
    if ($dh = opendir($dir))
    {
        while (($file = readdir($dh)) !== false)
        {
            if ($file == '.' or $file == '..')
            {
                continue;
            }
            if (is_file($dir . $file)) {
                $n++;
            }
        }
    }
    closedir($dh);
    return $n;
}

//通知在线的人
function notice($dir){
    if ($dh = opendir($dir))
    {
        while (($file = readdir($dh)) !== false)
        {
            if ($file == '.' or $file == '..')
            {
                continue;
            }
            if (is_file($dir . $file)) {
                $array[]=file_get_contents($dir.$file);
            }
        }
    }
    closedir($dh);
    return $array;
}
```

5.在同级目录下建立client文件，存放信息



PHP 即使通讯

1.socket.php 一样，使用时需要开启

```php
<?php
    //创建websocket服务器对象，监听0.0.0.0:9502端口
    $ws = new swoole_websocket_server("0.0.0.0", 9502);

    //监听WebSocket连接打开事件
    $ws->on('open', function ($ws, $request) {
        $fd[] = $request->fd;
        $GLOBALS['fd'][] = $fd;
        //$ws->push($request->fd, "hello, welcome\n");
    });

    //监听WebSocket消息事件
    $ws->on('message', function ($ws, $frame) {
        $msg =  'from'.$frame->fd.":{$frame->data}\n";
    //var_dump($GLOBALS['fd']);
    //exit;
        foreach($GLOBALS['fd'] as $aa){
            foreach($aa as $i){
                $ws->push($i,$msg);
            }
        }
       // $ws->push($frame->fd, "server: {$frame->data}");
        // $ws->push($frame->fd, "server: {$frame->data}");
    });

    //监听WebSocket连接关闭事件
    $ws->on('close', function ($ws, $fd) {
        echo "client-{$fd} is closed\n";
    });

    $ws->start();
```

2.socket.html聊天页面

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="msg"></div>
<input type="text" id="text">
<input type="submit" value="发送数据" onclick="song()">
</body>
<script>
    var msg = document.getElementById("msg");
    var wsServer = 'ws://60.205.208.176:9502';
    //调用websocket对象建立连接：
    //参数：ws/wss(加密)：//ip:port （字符串）
    var websocket = new WebSocket(wsServer);
    //onopen监听连接打开
    websocket.onopen = function (evt) {
        //websocket.readyState 属性：
        /*
        CONNECTING    0    The connection is not yet open.
        OPEN    1    The connection is open and ready to communicate.
        CLOSING    2    The connection is in the process of closing.
        CLOSED    3    The connection is closed or couldn't be opened.
        */
        msg.innerHTML = websocket.readyState;
    };

    function song(){
        var text = document.getElementById('text').value;
        document.getElementById('text').value = '';
        //向服务器发送数据
        websocket.send(text);
    }
      //监听连接关闭
//    websocket.onclose = function (evt) {
//        console.log("Disconnected");
//    };

    //onmessage 监听服务器数据推送
    websocket.onmessage = function (evt) {
        msg.innerHTML += evt.data +'<br>';
//        console.log('Retrieved data from server: ' + evt.data);
    };
//监听连接错误信息
//    websocket.onerror = function (evt, e) {
//        console.log('Error occured: ' + evt.data);
//    };

</script>
</html>
```

