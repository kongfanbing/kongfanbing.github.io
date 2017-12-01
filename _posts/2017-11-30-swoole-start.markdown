---
layout: post
title:  swoole初探
date:   2017-11-30 17:00:00 +0800
tags: [php,swoole,websocket]
---

>   学而不思则罔，思而不学则殆。
>
>   ——孔子



swoole的新手入门教程的开头是这么写的：

>- 熟练使用PHP语言
>- 熟练使用MySQL、Redis数据库
>- 熟练使用Linux操作系统
>- 基本了解Unix网络编程相关知识（参阅《Unix网络编程（卷1）》）
>- 基本的gdb使用

基于它的入门条件较高，所以我也不用多费篇幅去介绍它是什么，请自行领悟: [swoole官网](http://www.swoole.com)。

这次接触swoole，主要是想用它来实现websocket服务，所以代码也是以此为例。

同样基于它的入门条件，php的安装就不需要赘述了，直接进入swoole的安装步骤。

## swoole安装

使用`pecl install swoole`。

就是这么简单！如果有什么疑问，请参考我的另一篇文章[php安装扩展phpredis](https://kongfanbing.github.io/php/redis/2016/09/22/phpredis.html)以及[swoole官网wiki](https://wiki.swoole.com/)。

## websocket示例

服务端文件ws_server.php，执行`php ws_server.php`启动服务。

```php
<?php
//创建websocket服务器对象，监听0.0.0.0:9502端口
$ws = new swoole_websocket_server("0.0.0.0", 9502);

//监听WebSocket连接打开事件
$ws->on('open', function ($ws, $request) {
    var_dump($request->fd, $request->get, $request->server);
    $ws->push($request->fd, "hello, welcome\n");
});

//监听WebSocket消息事件
$ws->on('message', function ($ws, $frame) {
    echo "Message: {$frame->data}\n";
    global $ws;//调用外部的server
    // $server->connections 遍历所有websocket连接用户的fd，给所有用户推送
    foreach ($ws->connections as $fd) {
    	$ws->push($fd, $frame->fd." ".date("m-d H:i:s").":{$frame->data}");
    }
});

//监听WebSocket连接关闭事件
$ws->on('close', function ($ws, $frame) {
    echo "client-{$frame} is closed\n";
    global $ws;//调用外部的server
    // $server->connections 遍历所有websocket连接用户的fd，给所有用户推送
    foreach ($ws->connections as $fd) {
    	$ws->push($fd, $frame." ".date("m-d H:i:s")."已退出。");
    }

});

$ws->start();
?>
```

前端调用ws.html

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
<title>swoole-ws</title>
<script src="https://code.jquery.com/jquery-3.2.1.min.js"></script>
</head>
<body>
<textarea id="content"></textarea>
<button id="submit">提交</button>
<hr>
<div id="messages"></div>
</body>
<script>
var wsServer = 'wss://127.0.0.1/wss/';
var websocket = new WebSocket(wsServer);
websocket.onopen = function (evt) {
    $("#messages").append("<br />Connected to WebSocket server.");
};

$("#submit").on("click", function(){
	var content = $("#content").val();
	websocket.send(content);
})
websocket.onclose = function (evt) {
    $("#messages").append("<br />Disconnected");
};

websocket.onmessage = function (evt) {
    $("#messages").append('<br />' + evt.data);
};

websocket.onerror = function (evt, e) {
    $("#messages").append('<br />Error occured: ' + evt.data);
};
</script>
</html>
```

为了可以使用wss，需要修改web服务器配置，以nginx为例：

```nginx
server {
	listen       80;
	listen       443 ssl;
    server_name 127.0.0.1;
  	root /www/swoole/
    ssl    on;
    ssl_certificate    /etc/ssl/cert.pem;
    ssl_certificate_key    /etc/ssl/key.pem;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

	location /wss/
	{
			proxy_pass http://127.0.0.1:9502;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header Host $host;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
			rewrite /wss/(.*) /$1 break;
			proxy_redirect off;
	}
}
```

## 结语

到这里应该结束了，可是在测试的时候，本想来一场手机和PC的“对话”，结果发现苹果手机的safari浏览器不能使用wss，ws正常。初步推断是证书不匹配的问题，如果是 最好啦，但如果不是的话，还得再排查，所以——

## 未完待续。。。

