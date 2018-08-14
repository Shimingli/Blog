---
title: Android 9 Pie 正式版总结
date: 2018-08-14 11:48:40
tags: [Android P,谷歌 ,Android 9 正式版]   
---


* 先声明下，以下的整理的资料一部分来源于公众号`谷歌开发者`,一部分来源于[谷歌安卓官方网站](https://developer.android.google.cn/)（不用翻墙，gogogo），还有小部分自己的理解。
![Android 9](https://upload-images.jianshu.io/upload_images/5363507-88fbec19dcb39405.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
*  微信公众号号`谷歌开发者`推送了 `Android 9 Pie 现已面向全球正式发布！`,就去更新 `Android API 28`,想看看正式版那些功能，就发生了如下的故事 


![钉钉工作群](https://upload-images.jianshu.io/upload_images/5363507-246697c1c7a1fd7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这是我的电脑`IP`地址
![我的电脑IP.png](https://upload-images.jianshu.io/upload_images/5363507-2570704afb0266ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![和领导的聊天记录](https://upload-images.jianshu.io/upload_images/5363507-981c91a3d7305557.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*  然后我的电脑就被限制速度了，为了同事们能够好好工作，现在模拟器镜像都不能下载了！哈哈

![](https://upload-images.jianshu.io/upload_images/5363507-4fdf1483b6f645f7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*  虽然不能完成`Android 9`有些 `api`的具体的效果，先mark下，后续再来添上

*  今天我终于把`API 28`的源码下载下来了。哈哈！
##动态电量管理
####  1、应用待机分组
 *  Android P 新增加应用待机分组的功能，让系统根据用户的使用情况而限制应用调用CPU或网络等设备的资源, 应用待机分组是 Android P 新添加的一项电量管理功能，它能根据应用的使用频率或者最近一次使用时间，对其资源请求进行优先级排序！
 *  优先分组   在默认的情况下，系统会更具应用的近期的使用的情况进行等级划分，应用活跃度越高，所处的分组优先级就越高，也更加容易获取设备的资源；第二种是系统安装了通过利用机器学习预加载的应用，从而预测各个应用的使用概率，然后将他们编配值相应的群组，在我朝手机设备厂商应该不会是这种。设备厂商可以自行规定非活跃应用的群组划分规则(我朝特色，厂商白名单)。请开发者不要试图篡改应用所处的群组，而是专注于改善应用行为，确保应用被划分至目标群组后，依旧能够顺利运行（小公司，好好提升技术，）
##### 5个分组如下  活跃 (Active)、工作 (Working set)、 常用 (Frequent)、 极少 (Rare)、应用偶尔被使用 (App is not frequently used)
  *   `活跃 (Active)`: 应用正在被使用 （每个应用都可以）
       1、启动了一个Activity
       2、正在运行的前台服务
        3、另外一个应用已关联该应用  
      4、 用户点击了推送

  *  ` 工作 (Working set)`: 应用使用频率很高
 1、若应用的运行频率很高，但目前并未处于“活跃”状态，它就会被划分至工作群组，例如用户常用的社交媒体应用。此外，该群组还包括了那些被间接使用的应用。微信 QQ 支付宝 ，工作分组内的应用任务（Job）运行和闹铃受到系统的部分限制

  *   `常用 (Frequent)`: 应用经常但不是每天被使用
     常用应用指用户经常使用但不是每天使用的应用，比如用户在健身房使用的打卡应用可能就属于这一群组。跑步的APP系统对常用分组采用的限制更强，应用运行任务(job)和触发闹铃的能力都会受到影响，而且接受的高优先性FCM消息也有数量上限


  *  `  极少 (Rare)`: 应用偶尔被使用
         若应用的使用频率很低，它就会被划分至该分组，(比如说你去某个地方，订酒店之类的APP，)该群组下的应用在任务 (job)、闹铃和高优先性FCM消息的资源调用上都会受到严格的限制。此外，网络访问能力也会受到影响

  *   `应用偶尔被使用 (App is not frequently used)`
    安装后一次都未被使用过的应用将被划分至 “从不” 这一特殊群组，并受到十分严格的系统限制  我们自己的手机上有这种的App 很多

 *  我们做应用层`APP`应该怎么应对这几种分组 ？    
    *  每个模式下，都能打开APP，确保App不能炸掉
    *  要确保有启动的 Launcher Activity，如果没有的话，有可能你的应用不会切换到活跃分组
    *  推送的消息要具有可操作性，这个的意思就是说，点了通知栏要跳到应用去
    *  若应用在接受高优先级的 FCM 消息之后未能发送推送，用户将无法与应用产生互动并将其优先级提升至 “活跃” 等级。
    *  如果用户多次忽略某个App的推送，系统会去询问用户是否不再接受此推送 。所以不要乱去推送，为了保持活跃群组!

*  后台的限制 （微信经常这样）当系统检测到应用消耗过多的资源时，系统会发通知询问用户是否需要限制该应用的后台的活动
1、第一中期情况是，频繁使用唤醒锁 (wake locks)：屏幕关闭后，局部唤醒锁 (Partial wake lock) 连续开启 1 小时；
  2、过多的后台服务：当应用目标 API 等级低于 26，且运行过多后台服务。

* Android P 进一步提升了省电模式的性能。需要由设备的厂商来决定采用的具体的限制
作为开发者 我们自己，我们需要在省电模式下测试应用。确保自己的应用能够安全的上线和运行 
```
        //使用 Android Debug Bridge 命令 https://mp.weixin.qq.com/s?__biz=MzAwODY4OTk2Mg==&mid=2652046811&idx=1&sn=f0340e6fabb07a3ee40db45bdd58e7b0&chksm=808ca59eb7fb2c883c6ae99be7c84460f48886cd79bb0de886a5bac84afa2d8050a58339cc89&scene=21#wechat_redirect
```

####  2、后台限制
*  `Android P `新增后台限制功能，如果应用出现 `Android Vitals` 里面所描述的行为，系统将提醒用户限制该应用的访问设备的资源!
* `Android Vitals`:是谷歌提高Android设备稳定性和性能的一项举措。当选择的用户运行你的应用程序时，他们的Android设备记录各种度量，包括关于应用程序稳定性、应用程序启动时间、电池使用时间、渲染时间和权限拒绝的数据。谷歌播放控制台聚集这些数据并将其显示在Android虚拟仪表板中。仪表板突出了崩溃率、ANR率、过度唤醒和卡尾锁：这是开发人员应该关注的核心。所有其他的vitals，当适用于你的APP或游戏类型时，都应该被监控，以确保它们不会产生负面影响。如果产生了，应用的商店被发现的可能性低，说到底的意思就是，垃圾应用，我不帮你推荐!

####   3、省电模式的优化  
   *  Android P 优化了现在的省电助手的功能，在启动该功能后，系统将对所有的后台运行实施加以限制

####  4、低耗能模式， 当用户一段时间没有使用设备时候，设备将进入低耗电模式，所有的应用都将要受到影响。`Android P `并没有针对低电耗模式做出改变.
* 低耗电模式:   低电耗模式通过在设备长时间处于闲置状态时推迟应用的后台 CPU 和网络 Activity 来减少电池消耗。应用待机模式可推迟用户近期未与之交互的应用的后台网络 Activity. 如果用户设备未插接电源、处于静止状态一段时间且屏幕关闭，设备会进入低电耗模式。 在低电耗模式下，系统会尝试通过限制应用对网络和 CPU 密集型服务的访问来节省电量。 这还可以阻止应用访问网络并推迟其作业、同步和标准闹铃.一旦用户通过移动设备、打开屏幕或连接到充电器唤醒设备，系统就会立即退出低电耗模式，并且所有应用都将返回到正常 Activity。
* 有个例子。手机设备8.0上，打开视频APP下载视屏，关闭屏幕，一会视屏App就会关闭，在以前的版本不会出现，这就是低电耗模式!

#### 5、Slices  App Actions
*   Google 想要透过` Action `和 `Slice` 这两个功能让使用者减少操作动作，让应用程式和操作行为更紧密结合，把以前需要操作四五下才能完成的事减少到操作两三下或是操作一下就能自动完成。
*  `App Actions `是一种全新的应用推荐的方式，它能够对应用语义意图和使用场景进行分析，并根据分析结果在适当的时机向用户推荐应用,比如说在用户插入耳机的时候，推荐开启 音乐`App` 等等 --->谷歌没有公布使用的方式  
* `Slice`: 的概念则是 Google Assistant 的延伸，让使用者能快速使用到 app 里的某个特定功能，只要开发者导入 Slice 功能，使用者在使用搜寻、Google Play 商店、Google Assitant 或其他内建功能时都会出现 Slice 的操作建议。在`go`语言中 有个叫切片` slice `，它的意思是：在初始定义数组时，我们并不知道需要多大的数组，因此我们就需要“动态数组”。在Go里面这种数据结构叫`slice`.

####  6、[文本识别与 Smart Linkify](       https://developer.android.google.cn/reference/android/view/textclassifier/package-summary)     
* 在 Android 9 中，我们对识别文本的机器学习模型进行了扩展，使其可以借助 TextClassifier API 识别出类似日期或航班号这样的信息。此外， Smart Linkify 允许开发者通过 Linkify API 使用文本识别模块完成多项操作，比如对用户可采取的操作提出建议。Smart Linkify 让系统在文本识别精确度与速度上都有明显的提升。
*  TextClassifier  后续更新下Demo  研究中
![demo.gif](https://upload-images.jianshu.io/upload_images/5363507-085482fa43bb2721.gif?imageMogr2/auto-orient/strip)

```
        TextClassifier API
       https://developer.android.google.cn/reference/android/view/textclassifier/package-summary
      Linkify API
     https://developer.android.google.cn/reference/android/text/util/Linkify
```
#### 7、[神经网络 API 1.1]( https://developer.android.google.cn/ndk/guides/neuralnetworks/index.html) 
* `Android 9.0` 对神经网络 API 进行了扩展与改进，进一步优化 Android 对机器学习硬件加速的支持。神经网络 API 1.1 共增加了对 9 个新算子的支持，它们分别是` Pad、BatchToSpaceND、SpaceToBatchND、Transpose、Strided Slice、Mean、Div、Sub 和 Squeeze。TensorFlow Lite `就是一个已经用上此 API 的典型机器学习框架。


## 人机交互

#### 1、全新系统导航
*  ` Android 9 `迎来了全新的系统导航，让多任务切换及关联应用探索变得更加简单。您只需要向上滑动屏幕就可以全屏预览最近使用过的应用，轻触预览页后便可以切换至所选应用。

![全新系统导航.gif](https://upload-images.jianshu.io/upload_images/5363507-57240ac05547289b.gif?imageMogr2/auto-orient/strip)

#### 2、 [凹口屏支持]( https://developer.android.google.cn/guide/topics/display-cutout/)  
*   想起来 苹果大佬 还是牛逼，刘海是为了屈服某种硬件的缺陷，然后谷歌就开始支持了
* ` Android 9` 中加入了凹口屏支持，让您的应用可以充分利用最新全面屏，展现应用的独特魅力。该功能可以在大部分应用中无缝工作，系统会通过调整状态栏高度将应用内容与屏幕缺口区域分开。如果您的应用含有沉浸式内容，您可调用` display cutout APIs` 确认缺口形状与位置，然后请求围绕缺口进行全屏布局。另外，我们还加入了开发者选项来模拟任意设备上的凹口形状，从而极大简化了应用支持凹口屏幕所需的构建以及测试流程。
#### 3、 通知与智能回复
*   `Android 9` 进一步改善了通知的实用性与可操作性。消息类应用可以调用新的 `MessagingStyle API` 来显示对话，附加照片和表情，或者提供智能回复建议。
```
     // Create new Person.  完蛋了，这下在安卓中Person类有了，不仅仅是一个了
        val sender = Person()
                .setName(name)
                .setUri(uri)
                .setIcon(null)
                .build()
      // Create image message.
        val message = Notification.MessagingStyle.Message("Picture", time, sender)
                .setData("image/", imageUri)
        val style = Notification.MessagingStyle(getUser())
                .addMessage("Check this out!", 0, sender)
                .addMessage(message)
```

#### 4、[文本放大镜]( https://developer.android.google.cn/reference/android/widget/Magnifier)
*  Android 9 中添加文字放大镜工具 (Magnifier widget)，以提升文本选择方面的用户体验。由于该放大器提供了可以在文本上方拖拽的文本放大面板，所以有助于用户精准地定位光标或文本选择手柄。该功能可以灵活运用在所有附加在窗口的视图上，个性化小部件和定制文本呈现均是不错的应用场景。
*  该放大器工具还可以提供任何视图或界面的放大版本，而不仅仅是文本。
![文本放大.gif](https://upload-images.jianshu.io/upload_images/5363507-1298304aefb3084e.gif?imageMogr2/auto-orient/strip)

## 用户安全与隐私
#### 1、统一身份验证对话框 

  *  为了保障用户在不同甘银强和应用之间能够获得一致的体验，Android P 引入了统一的身份验证对话框，提醒用户进行操作。应用可以不自行的设计，该API还支持面部识别` 虹膜识别技术 ` 是基于眼睛中的虹膜进行身份识别，应用于安防设备（如门禁等），以及有高度保密需求的场所。
  
     *  优点
 1．便于用户使用；
  2．可能会是最可靠的生物识别技术；.
 3．不需物理的接触；
  4．可靠性高。
     *   缺点
 1．很难将图像获取设备的尺寸小型化；
  2．设备造价高，无法大范围推广；
  3．镜头可能产生图像畸变而使可靠性降低；
  4．两大模块：硬件和软件；
  5．一个自动虹膜识别系统包含硬件和软件两大模块：虹膜图像获取装置和虹膜识别算法。分别对应于图像获取和模式匹配这两个基本问题。

* 代码如下
```
 // TODO 一定要在 API 28的模拟器上跑 要不然 app 会直接奔溃掉
        val BiometricPrompt = BiometricPrompt.Builder(this)
                .setTitle("指纹验证")
                .setDescription("描述")
                .setNegativeButton("取消", mainExecutor, DialogInterface.OnClickListener { dialogInterface, i -> Log.i(UserSecurityAndPrivacyActivity@ this.localClassName, "Cancel button clicked") })
                .build()
        val mCancellationSignal = CancellationSignal()
        mCancellationSignal.setOnCancelListener {
            CancellationSignal.OnCancelListener {
                fun onCancel() {
                    println("取消了啊")
                }
            }
        }
        val mAuthenticationCallback = object : BiometricPrompt.AuthenticationCallback() {
            override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
                super.onAuthenticationError(errorCode, errString)
                println("发生了 错误了啊")
            }

            override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
                super.onAuthenticationSucceeded(result)
                println("发生了 成功了啊")
            }

            override fun onAuthenticationFailed() {
                super.onAuthenticationFailed()
                println("发生了 失败了")
            }
        }
        BiometricPrompt.authenticate(mCancellationSignal, mainExecutor, mAuthenticationCallback)
```

#### 2、[Android Protected Confirmation](https://developer.android.google.cn/preview/features/security#android-protected-confirmation)
*  安卓保护确认：运行Android 9或更高版本的支持设备使您有能力使用Android保护的确认。使用此工作流时，应用程序向用户显示提示，要求用户批准简短语句。这个声明允许应用程序重新确认用户希望完成一个敏感的交易，例如支付。

#### 3、`KeyStore` [加强密钥安全保护](https://link.juejin.im?target=https%25253A%25252F%25252Fdeveloper.android.google.cn%25252Fpreview%25252F)
* 加入了一个新的` KeyStore` 类 —— `StrongBox`，并提供相应的 API 来支持那些提供了防入侵硬件措施的设备，比如独立的 CPU，内存以及安全存储。

####4、 DNS over TLS 基于TLS的DNS
 *  大多的网络连接 --DNS查询--》返回IP地址--》客服端就能连接上网页了。互联网的大佬们一直在致力于新的NDS的协议的开发，该协议就是NDS over  TLS协议     `TLS`：安全传输层协议（TLS）用于在两个通信应用程序之间提供保密性和数据完整性
  * `Android  P` 正式版内置对 `DNS over TLS `的支持，在 “网络和互联网” 设置中添加了隐私 DNS (Private DNS) 模式。
*  NS over TLS 模式自动为所有系统上的应用提供安全 DNS查询。不过，若应用未使用系统 API，而是自行运行 DNS 查询，它们必须确保在系统进行安全连接情况下，不发送不安全的 DNS 查询。应用可以调用新的 API
```
 LinkProperties.isPrivateDnsActive()
```

#### 5、`Android P `默认使用了`HTTPS`

#### 6、基于编译器的安全缓解措施
 * `Android 9` 将进一步扩展编译器级别的安全缓解措施，借助运行时危险行为监测进一步加强平台安全建设。`Android 9 `通过控制流程完整性 `(CFI)` 技术解决了代码重用` (code-reuse) `和任意代码执行两大漏洞，并扩展了` CFI `在媒体框架和其它关键安全组件内的使用范围，如` NFC` 与蓝牙。同时，`Android 9 `还针对 `Android` 常见内核的 `LLVM` 编译添加了 `CFI `内核支持。

#### 7、用户隐私
* `Android 9 `新加入多项机制，进一步加强了对用户隐私的保护。系统禁止所有处于空闲状态的应用对话筒、摄像头和所有 `SensorManager `传感器的访问。当应用的 `UID` 空闲时，麦克风将会报告 “无音频信号”，传感器将会停止报告事件，应用使用的摄像头也会断开连接，并在应用试图访问时生成错误。在大多数情况下，这些限制不会对现有应用造成新的问题，谷歌建议从应用中移除此类传感器请求。
* `Android 9` 还让用户控制是否允许访问平台` build.serial `识别码 (它被 READ_PHONE_STATE 权限保护) ,设备的唯一标识`the hardware serial number`.
```
    //获取硬件序列号，如果可用的话。
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_PHONE_STATE) != PackageManager.PERMISSION_GRANTED) {
            //注意需要在 高版本的 SDK的手机上运行
            val serial = Build.getSerial()
        }
```
## 摄像和影音的全面升级

#### 1、多摄像头的API的改进 -->
* 从 Android 9 开始，您可以在支持多摄像头 API 的设备上通过两个或更多实体摄像头同时访问视频流；在配有双前置或双后置摄像头的设备上，实现单摄像头无法实现的创新功能：如无缝变焦、散景和立体视觉。该 API 还允许您调用可以在两台或更多台摄像头之间自动切换的逻辑或混合摄像头视频流。

#### 2、使用动态处理增强音频-->降噪技术
* iPhone手机的降噪技术：双麦克风降噪技术，即手机中内置的两个麦克风，一个保持清晰通话，另一个麦克风从物理上主动消除噪音，通过收集外界的噪声，运用内部算法进行处理后，发出与噪音相反的声波，利用抵消原理消除噪音。其他手机估计也差不多！
* 开发者可以调用 `Dynamics Processing API `对音频进行动态处理，通过分离出特定频率的声音，降低过大的音量，或者增强过小的音量，来改善应用的音频质量。比如说，即便说话者声音小，离麦克风远，而且外界环境十分嘈杂，您的应用依然可以有效捕捉并他/她的声音，并进行相应优化。该 API 提供了多声场、多频段的动态处理效果，包括一个预均衡器、一个多频段压缩器，一个后均衡器以及一个串联的音量限制器。
#### 3、ImageDecoder API 
*  `ImageDecoder` 通过一种更为简单的方式将图像解码为位图或 `drawable`。`ImageDecoder` 允许您从字节缓冲区、文件或 URI 创建位图或 drawable。它相比` BitmapFactory `有以下几个优势：支持精确缩放，支持单步解码至硬件存储器，支持解码后处理，以及动画图像解码。
* ` ImageDecoder`:一个的可以将`PNG, JPEG, WEBP, GIF, or HEIF`格式的图片的转换成`Drawable` 或者`Bitmap `对象的类。
  * 1、正常的使用:得到`drawable `
   ```
  val file =File("filename")
        val source = ImageDecoder.createSource(file)
        val drawable = ImageDecoder.decodeDrawable(source)
   ```
  * 2、传递`OnHeaderDecodedListener`，这里`ImageInfo`存放的是原始的图片的宽和高。可以修改用来修改图片宽高的时候修改`SampleSize`
   ```
  val listener = object : ImageDecoder.OnHeaderDecodedListener {
            override fun onHeaderDecoded(decoder: ImageDecoder, info: ImageDecoder.ImageInfo, source: ImageDecoder.Source) {
                decoder.setTargetSampleSize(2)
            }
        }
        val drawable1 = ImageDecoder.decodeDrawable(source, listener)
   ```

  * 3、如果解码的图片是`gif`，会被解码成`AnimatedImageDrawable`
   ```
    val drawable2 = ImageDecoder.decodeDrawable(source)
        if (drawable is AnimatedImageDrawable) {
            drawable.start()
        }
   ```

  * 4、解码出来的bitmap是不可变，可以使用`PostProcessor`来添加一些自定义的效果
   ```
    val drawable3 = ImageDecoder.decodeDrawable(source) { decoder, info, src ->
            decoder.setPostProcessor { canvas ->
                // This will create rounded corners.
                //创建圆角照片
                val path = Path()
                path.setFillType(Path.FillType.INVERSE_EVEN_ODD)
                val width = canvas.width
                val height = canvas.height
                 // 最低的API的要求是 21 所以我的工程里面的 API为 21
                path.addRoundRect(0.toFloat(), 0.toFloat(), width.toFloat(), height.toFloat(), 20.toFloat(), 20.toFloat(), Path.Direction.CW)
                val paint = Paint()
                paint.setAntiAlias(true)
                paint.setColor(Color.TRANSPARENT)
                paint.setXfermode(PorterDuffXfermode(PorterDuff.Mode.SRC))
                canvas.drawPath(path, paint)
                PixelFormat.TRANSLUCENT
            }
        }
   ```
  * 5、如果解码的照片是不完整的或者包含错误，解码的时候会抛出`DecodeException`，一些情况下，可能已经解码出一部分的照片，这个时候传递`OnPartialImageListener `，并返回`true`，就只显示解码出来的部分，剩余部分使用空白代替!
   ```
   val drawable5 = ImageDecoder.decodeDrawable(source) { decoder, info, src ->
            decoder.setOnPartialImageListener { e: ImageDecoder.DecodeException ->
                 true
            }
        }
   ```

## 网络连接与位置

#### 1、使用 Wi-Fi RTT ，进行室内定位
* `Android 9` 为` IEEE 802.11mc Wi-Fi `协议添加了平台支持 (也称为 Wi-Fi 往返时间，RTT)，这可以让您在应用中使用室内定位功能。在提供硬件支持的 Android 9 设备上，在启动位置服务并勾选 “允许获取地理位置信息” 选项后，应用就可以使用 RTT API 测量与附近 Wi-Fi 接入点 (AP) 的距离。设备不需要连接到 AP 便可以使用 RTT，而且为了保护隐私，只有手机能够确定距离，而 AP 不可以。
* 通过测量从设备到三个或更多 AP 的距离，您可以计算设备位置至 1 到 2 米的精度。这种精确度允许您创建更多新的体验：室内导航、基于位置的细粒度服务，例如，模糊语音控制 ( "打开这里的灯" ) ；以及基于位置的资讯服务 ( "这个产品有优惠活动吗？" )。

#### 2、JobScheduler 中的数据费用敏感度 
*  在 `Android 9 `中，`JobScheduler` 可以更好地帮助用户处理与网络相关的任务，并与运营商单独提供的网络状态信号相协调
*  `JobScheduler `是 `Android` 的一项核心服务，它可以帮助您针对低耗电模式、应用待机模式以及后台限制，妥善进行各种任务的调度。在 `Android 9 `中，`JobScheduler` 可以更好地帮助用户处理与网络相关的任务，并与运营商单独提供的网络状态信号相协调。任务现在可以声明预估数据量、信号预取以及指定详细的网络要求 —— 运营商可以报告网络状况是拥塞还是不计量，然后 `JobScheduler `会根据网络状态管理作业。例如，当网络拥塞时，`JobScheduler` 可能推迟大型网络请求；而在网络可以不计量使用时，则可以运行多种预加载作业 (例如，预读标题) 来改进用户体验。
####  3、 用于 NFC 支付和安全交易的 Open Mobile API
*   `Android 9` 将 `GlobalPlatform Open Mobile API` 的实现添加至平台中。在支持的设备上，应用可以使用 OMAPI API 访问安全元素 (SE) ，以启用智能卡支付等安全服务。硬件抽象层 (HAL) 提供了必要的 API，用于枚举多种可用的 Secure Elements (如 eSE, UICC 等)。


## 更强劲的性能表现

#### 1、ART 性能提升
*  ART是什么？
 * `AOT`是”Ahead Of Time”的缩写，指的就是ART(Anroid RunTime)这种运行方式。
* `JIT`是运行时编译，这样可以对执行次数频繁的dex代码进行编译和优化，减少以后使用时的翻译时间，虽然可以加快Dalvik运行速度，但是还是有弊病，那就是将dex翻译为本地机器码也要占用时间，所以Google在4.4之后推出了ART，用来替换Dalvik。
 * ART的策略与Dalvik不同，在ART 环境中，应用在第一次安装的时候，字节码就会预先编译成机器码，使其成为真正的本地应用。之后打开App的时候，不需要额外的翻译工作，直接使用本地机器码运行，因此运行速度提高。
  * 当然ART与Dalvik相比，还是有缺点的。
     * ART需要应用程序在安装时，就把程序代码转换成机器语言，所以这会消耗掉更多的存储空间，但消耗掉空间的增幅通常不会超过应用代码包大小的20%
     * 由于有了一个转码的过程，所以应用安装时间难免会延长
      * 但是这些与更流畅的Android体验相比而言，不值一提。
*   `Android 9` 借助 `ART` 运行时显著提高了应用的性能表现与运行效率。扩展了 `ART `对执行特征的使用，以优化应用并减少已编译应用代码的内存占用量。ART 现可使用特征文件信息在设备上重写 DEX 文件，帮助多个常见应用的内存占用减少高达 11％。我们期望借此减少系统 DEX 内存使用量并加快应用启动时间。


#### 2、Kotlin 优化
*    Kotlin 是 Android 开发的一等编程语言，改进了一些编译器优化，尤其是那些针对循环的编译器优化，以实现更好的性能。


