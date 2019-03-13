---
title: 我的第一个正式的Java后端程序（二）
date: 2019-03-13 17:05:09
tags:
---

* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 开发背景，公司新零售终端Pos系统从有赞战略转换为美团，但是门店还是原来的门店，但是我的大爷就不是原来的大爷，撸起袖子开干

* 期间我们物业方吹得有点紧，哈哈，这次主要原因是我太忙了，我要画原型图、要开发Pos系统、要升级店铺工具，使用内置打印机、要去新门店装机器、有时候还得关心门店的物料问题，最可怕我还得和供应商联系采购硬件设备，所以啊对不住，在博客里面说，所以一直拖着，真的是太忙了
![image.png](https://upload-images.jianshu.io/upload_images/5363507-2f4d9a0652ff2a90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--  more  -->


####  创建工程 
* 我犯了一个小小的错误，创建的工程名为`pp.id=retail-push-order-pafc` 这种格式的，但是我在远端的工程创建了这样的格式`pp.id=retail-pushorder-pafc`,这样就会在第一次会给第一次创建Jenkins索引的时候找不到，就会创建jar失败。希望下次注意
* 代码和以前差不多，唯一不同的区别就是通过`mq`来接受消息，然后我处理每一条消息然后组装成新的Json然后给推送过去,其他具体的可以看第一个工程 [的第一个正式的Java后端程序（一）](https://www.jianshu.com/p/8ff6c13ba45f),在这里我只写不同的地方
* 需要加的东西
```
#rocketmq
rocketmq.namesrv-addr=
#zookeeper
elasticjob.zk.serverLists
eureka.client.serviceUrl.defaultZone

#其他的配置
spring.application.name=retail-pushorder-pafc
server.port=8194
rocketmq.producer-group-name=retail-retail-push-order-pafc-producer
rocketmq.consumer-group-name=retail-retail-push-order-pafc-consumer

```
*rocketmq 消息队列作为高并发系统的核心组件之一，能够帮助业务系统解构提升开发效率和系统稳定性 可以看这篇文章 [Rocketmq原理&最佳实践](https://www.jianshu.com/p/2838890f3284)




####  其他的Config
* 一般使用了mq工程中肯定需要一个公用配置，这样才能使用
* 由于需要测试id和正式id不一样，而且我还不能去经常改动代码，所以需要一个WebConfig去判断目前在那个环境下，这样同一套代码就可以在联调环境和正式环境使用

![image.png](https://upload-images.jianshu.io/upload_images/5363507-0c60840fe10db39f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* `mq`有`consumer、producer`：一个是消费者，一个生产者，目前我自己的模块就是消费者，无需生产者，但是这个消息也是其他模块给我发送过来的

* 我们发包的流程
   * 上传代码到`git`厂库，合并代码到`mater`分支,打`Tag`，`Tag`的格式一般是`V1.0.0A01_20190312` 或者是正式包 `V1.0.0R01_20190312`，然后告诉测试你的分支和`Tag`，他们构建 `jar` ,然后测试---上线。

####  遇到的问题

* 遇到问题一：
  * 背景：`vpn`，去年我在做物流模块的时候，在公司的电脑，是不需要什么`vpn`，今年过来我还不知道这个事情，由于dev环境没有和外部联调的环境好多没有打通，导致我在`dev`调试不可能（就是我们收银机使用的是第三方系统的，第三方把订单给推送过来），我就必须去`qa`环境调试，万万没有想到，在`qa`环境也需要`vpn`，这个问题卡了我好几天，就差点卸载`ide`了，我肯定问了别人！
![image.png](https://upload-images.jianshu.io/upload_images/5363507-b95e3373aaca6b81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  * 还要运维给vpn账号和密码，然后登陆上去，才可以读取`qa`环境的`apollo`的配置
* 到这一步了
![image.png](https://upload-images.jianshu.io/upload_images/5363507-8ab26c107d32b6d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



* 遇到问题二：`qa`没有日志的输出
  * 原因是我在本地的跑了项目，qa服务器也跑了项目，所以我这边没有收到，后续关闭掉项目就可以了


* 遇到问题三：
    * 抛出一个问题         `float floatPrice= 9/100;  System.out.println(floatPrice); `结果为`0.0  `如果要想得到结果 那么就得 强制转换一下    `float floatPrice= (float) 9/100;
 System.out.println(floatPrice);` 我们项目中使用的是分，但是对方系统接收的元，所以需要转换一下单位，就是Java的基础知识了

![image.png](https://upload-images.jianshu.io/upload_images/5363507-79c83214d4a95f42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* 遇到问题四：
  * 线上环境Apollo读取不到，解决办法是在线上Apollo环境也要建一个和项目名称一样的Apollo配置，虽然我去读取的是一个通用的配置，但是也得创建这个
![image.png](https://upload-images.jianshu.io/upload_images/5363507-dda40674a5c19b4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 遇到问题五：
  *  收不到mq消息，其他配置都没有问题，那是因为需要手动创建`Topic`,其实它可以自动的创建，也可以手动，qa环境是自动的创建，线上环境是手动的，这个我们公司的运维还不知道，弄了几个小时，哈哈

![image.png](https://upload-images.jianshu.io/upload_images/5363507-b9cb156c44057cdf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* 遇到问题六：
  * 把所有的消息都给推送过去了，应该有`9`万左右的订单数据，等`Topic`创建，一下就推送了全部，本来就想推送今天的订单，但是历史的订单也一下全部的推送过去了


####  发包上线
* 申请上线
 ![image.png](https://upload-images.jianshu.io/upload_images/5363507-9af9f70b1e5eec0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 看到推送成功的消息，ok，收工
![image.png](https://upload-images.jianshu.io/upload_images/5363507-cc9a320a877ac54b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* **最后感谢勋哥和王宏大佬的鼎力帮助，还有感谢架构师老郭的指点，运维张渝的帮忙**



 