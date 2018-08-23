---
title: Go语言实现的WebSocket
date: 2018-07-15 16:02:08
tags: [Go,WebSocket]
---
* 最终的效果如下
<!--  more  -->
![Web端上传的信息](https://upload-images.jianshu.io/upload_images/5363507-1df3bbfdd7b78fff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Web端得到的打印的信息](https://upload-images.jianshu.io/upload_images/5363507-394f230f0c32cdab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![服务端的代码的实现](https://upload-images.jianshu.io/upload_images/5363507-1c4e57241edf86c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![服务端的信息](https://upload-images.jianshu.io/upload_images/5363507-7335f93af21b8ca3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。WebSocket通信协议于2011年被[IETF](https://baike.baidu.com/item/IETF)定为标准RFC 6455，并被RFC7936所补充规范。WebSocket是`HTML5`的重要特性，它实现了基于浏览器的远程`socket`，它使浏览器和服务器可以进行全双工通信，许多浏览器`Firefox、Google Chrome和Safari`都已对此做了支持
* 在`WebSocket`出现之前，为了实现即时通信，采用的技术都是“轮询”，即在特定的时间间隔内，由浏览器对服务器发出HTTP Request，服务器在收到请求后，返回最新的数据给浏览器刷新，“轮询”使得浏览器需要对服务器不断发出请求，这样会占用大量带宽。
* 很多的游戏的服务都是采用的是`Socket`，因为HTTP协议相对而言比较耗费性能， 随着`HTML5`的发展，`WebSocket`也逐渐发展成为很多页游公司接下来开发的一些手段。
* 安卓推送的原理： C2DM 推送 (Google) C2DM 推送简介 : 全称 Cloudto Device Messaging, Google 提供的 推送解决方案;
     * 运行方式 : 提供一个轻量级机制, 允许服务器通知应用程序, 主动与客户端进行数据交互, 处理消息排队, 并向运行于目标设备的应用程序分发消息;
     * 优点 : Google 提供的原生框架, 无需在应用中添加第三方代码 和 部署服务器端;
     * 缺点 : 1.该推送依赖 Google 服务器, 需要绑定 Google 帐号, 目前在中国 Google 被屏蔽, 无法使用; 2. 许多手机厂商去掉了软件中的该模块;
* 极光推送的原理：因为` IP v4` 的 IP 量有限，运营商分配给手机终端的 IP 是运营商内网的 IP，手机要连接 Internet，就需要通过运营商的网关做一个网络地址转换`Network Address Translation，NAT`。简单的说运营商的网关需要维护一个外网 IP、端口到内网 IP、端口的对应关系，以确保内网的手机可以跟 `Internet` 的服务器通讯。

* Android 平台上长连接的实现
  * Timer
Android 的 Timer 类可以用来计划需要循环执行的任务，Timer 的问题是它需要用 WakeLock 让 CPU 保持唤醒状态，这样会大量消耗手机电量，大大减短手机待机时间。
  *  AlarmManager 这篇文章有介绍怎么使用`AlarmManager`[安卓网络和电量优化](https://www.jianshu.com/p/82b76e0cb41e)
AlarmManager 是 Android 系统封装的用于管理 RTC 的模块，RTC (Real Time Clock) 是一个独立的硬件时钟，可以在 CPU 休眠时正常运行，在预设的时间到达时，通过中断唤醒 CPU。
这意味着，如果我们用 AlarmManager 来定时执行任务，CPU 可以正常的休眠，只有在需要运行任务时醒来一段很短的时间。极光推送的 Android SDK 就是基于这种技术实现的。[极光官方文档](http://blog.jiguang.cn/jpush_wireless_push_principle/)

* WebSocket URL的起始输入是`ws://`或是`wss://`（在SSL上）。一个带有特定报头的HTTP握手被发送到了服务器端，接着在服务器端或是客户端就可以通过JavaScript来使用某种套接口（socket），这一套接口可被用来通过事件句柄异步地接收数据。


####  WebSocket 原理 
 * WebSocket的协议:在第一次`handshake`通过以后，连接便建立成功，其后的通讯数据都是以”\x00″开头，以”\xFF”结尾。在客户端，这个是透明的，`WebSocket`组件会自动将原始数据“掐头去尾”。

![request的信息.png](https://upload-images.jianshu.io/upload_images/5363507-edfd9eb6a6232a99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
GET http://localhost:8080/shiming HTTP/1.1
Host: localhost:8080
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Origin: file://
Sec-WebSocket-Version: 13
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36 SE 2.X MetaSr 1.0
Accept-Encoding: gzip, deflate, sdch, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: _ga=GA1.1.27955907.1529919744
Sec-WebSocket-Key: PCD+pA79juC6tlBK9zD3Vw==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits

```
* `Sec-WebSocket-Key`这个是个随机的值，是一个经过base64编写后的数据`Sec-WebSocket-Key: PCD+pA79juC6tlBK9zD3Vw==`
* 然后服务器收到这个请求之后把这个字符串连接上一个固定的字符串 `258EAFA5-E914-47DA-95CA-C5AB0DC85B11` 为啥是这样的，我目前还不明白，仅仅是自己记录下而已。
* 最终得到:
```
PCD+pA79juC6tlBK9zD3Vw==258EAFA5-E914-47DA-95CA-C5AB0DC85B11
```
* 对该字符使用shal 安全散列算法计算出二进制的值，然后用base64 对其进行编码，即可以得到握手的字符串: ` 8+5CLWTBYLARKoxBxS5uk6s6zZo=` ,如下图所示

![response信息.png](https://upload-images.jianshu.io/upload_images/5363507-174d684336e9c839.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 8+5CLWTBYLARKoxBxS5uk6s6zZo=

```
* `Sec-WebSocket-Accept`作为响应头的值反馈给客户端。

#### Go语言实现Websocket 
* 由于Go语言标准包里面没有对`WebSocket`的支持，但是官方维护的`go.net`对这个有支持，所以可以获取
```
go get golang.org/net/websocket
```


* 但是有个小问题，当我 ` go get`后，我在代码中导入包会报错，同时去掉x也不行，所以我在本地目录创建了一个x的目录，然后把`net`全部放进去了 
![注意问题.png](https://upload-images.jianshu.io/upload_images/5363507-b8bb98e7cdda07ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![导包](https://upload-images.jianshu.io/upload_images/5363507-952095a76359642d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* html 代码
```
<html>
<head>
    <title>好好学习</title>
</head>
<body>
<script type="text/javascript">
    var sock = null;
    // var wsuri = "wss://127.0.0.1:8080"; //本地的地址 是可以改变的哦
     var wsuri = "ws://localhost:8080/shiming"; //本地的地址 是可以改变的哦


    window.onload = function() {
        //可以看到客户端JS，很容易的就通过WebSocket函数建立了一个与服务器的连接sock，当握手成功后，会触发WebScoket对象的onopen事件，告诉客户端连接已经成功建立。客户端一共绑定了四个事件。
        console.log("开始了 onload");

        sock = new WebSocket(wsuri);
        //建立连接后触发
        sock.onopen = function() {
            console.log(" 建立连接后触发 connected to " + wsuri);
        }
        // 关闭连接时候触发
        sock.onclose = function(e) {
            console.log("关闭连接时候触发 connection closed (" + e.code + ")");
        }
        // 收到消息后触发
        sock.onmessage = function(e) {
            console.log("收到消息后触发 message received: " + e.data);
        }
        //发生错误的时候触发
        sock.onerror=function (e) {
            console.log("发生错误时候触发"+wsuri)
        }
    };
     //如果sock被关闭掉了 这里 也会报错的啊
    function send() {
        var msg = document.getElementById('message').value;
        sock.send(msg);
    };
</script>
<h1>GoWebSocketDemo</h1>
<form>
    <p>
        Message: <input id="message" type="text" value="你好啊  shiming 小哥哥  嘿嘿   ">
    </p>
</form>
<button onclick="send();">给服务器发送消息</button>
</body>
</html>
```
* go 代码
```
package main

import (
	"fmt"
	"net/http"
    //草拟吗 自己创建的目录 哈哈哈哈哈    还好我比较聪明  要不然 就完蛋了  麻痹
	"golang.org/x/net/websocket"
	"log"
)

func main() {
	fmt.Println("Go语言标准包里面没有提供对WebSocket的支持，但是在由官方维护的go.net子包中有对这个的支持 go get golang.org/x/net/websocket")
	//打印这个信息就，os.Exit(1)  退出程序
	//log.Fatal("shiming")  todo  草拟吗 啊   看清楚啊   后面的域名的地址 有个老子的名字啊
    http.Handle("/shiming",websocket.Handler(Echo))
	 if err:=http.ListenAndServe(":8080",nil);err!=nil{
	 	log.Fatal(err)
	 }


}

func Echo(w *websocket.Conn)  {
    var error error
	for   {
		var reply string
		if  error= websocket.Message.Receive(w,&reply);error!=nil{
			fmt.Println("不能够接受消息 error==",error)
			break
		}
		fmt.Println("能够接受到消息了--- ",reply)
		msg:="我已经收到消息 Received:"+reply
		//  连接的话 只能是   string；类型的啊
		fmt.Println("发给客户端的消息： "+msg)

		if error = websocket.Message.Send(w, msg); error != nil {
			fmt.Println("不能够发送消息 悲催哦")
			break
		}
	}
}
```

* 说明一点：`http.Handle("/shiming",websocket.Handler(funName))`,如果在这里有路由的话，记得在  `html`中也要改成一样的, html中的代码 ：` var wsuri = "ws://localhost:8080/shiming"`
* `x`目录自己创建一个，把`net`包剪切进去就可以






