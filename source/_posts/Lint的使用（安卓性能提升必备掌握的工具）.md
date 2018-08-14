---
title: Lint的使用（安卓性能提升必备掌握的工具）
date: 2018-05-05 16:24:21
tags: Lint的使用
---
## 什么是 Lint

Lint 是Android Studio 提供的 **代码扫描分析工具**，它可以帮助我们发现代码结构/质量问题，同时提供一些解决方案，而且这个过程不需要我们手写测试用例。

Lint 发现的每个问题都有描述信息和等级（和测试发现 bug 很相似），我们可以很方便地定位问题，同时按照严重程度进行解决。

当然这个“严重程度”我们可以手动调节，有些原则问题不容侵犯，必须提升到 error，而有的个别问题也可以无视，毕竟人非圣贤孰能无过嘛。
<!--  more  -->
## Lint 工作方式简单介绍

Lint 会根据预先配置的检测标准检查我们 Android 项目的源文件，发现潜在的 bug 或者可以优化的地方，优化的内容主要包括以下几方面：

*   Correctness：不够完美的编码，比如硬编码、使用过时 API 等
*   Performance：对性能有影响的编码，比如：静态引用，循环引用等
*   Internationalization：国际化，直接使用汉字，没有使用资源引用等
*   Security：不安全的编码，比如在 WebView 中允许使用 JavaScriptInterface 等
*   …
## Android Studio 中使用 Lint
1、
![image.png](https://upload-images.jianshu.io/upload_images/5363507-92ee4366d54c50b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2、
![image.png](https://upload-images.jianshu.io/upload_images/5363507-153c7217f041a5c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)/3、
![image.png](https://upload-images.jianshu.io/upload_images/5363507-6f14d8059a5c2fd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 拿一个以前的项目的分析结果

![微信图片_20180503210246.png](https://upload-images.jianshu.io/upload_images/5363507-57cec658c6cdd634.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210312.png](https://upload-images.jianshu.io/upload_images/5363507-5e4fbd4975e4c859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210315.png](https://upload-images.jianshu.io/upload_images/5363507-067fafdd49c2c1ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210319.png](https://upload-images.jianshu.io/upload_images/5363507-828b47ade1709a9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210324.png](https://upload-images.jianshu.io/upload_images/5363507-688c16f15234094e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210328.png](https://upload-images.jianshu.io/upload_images/5363507-d072a52eee950018.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210331.png](https://upload-images.jianshu.io/upload_images/5363507-5968e53780fc9dc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210334.png](https://upload-images.jianshu.io/upload_images/5363507-9ede476901b5a0b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210337.png](https://upload-images.jianshu.io/upload_images/5363507-4a062ded33a100ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210342.png](https://upload-images.jianshu.io/upload_images/5363507-ba396d5266a7121e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210345.png](https://upload-images.jianshu.io/upload_images/5363507-09a04ff9a27664fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_20180503210427.png](https://upload-images.jianshu.io/upload_images/5363507-327b0a6db096ef9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_201805032104272.png](https://upload-images.jianshu.io/upload_images/5363507-6e4d2db0c47ada54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_201805032104273.png](https://upload-images.jianshu.io/upload_images/5363507-307f1fc85893e20f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_201805032104274.png](https://upload-images.jianshu.io/upload_images/5363507-3eb30fb8b8463717.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_201805032104275.png](https://upload-images.jianshu.io/upload_images/5363507-287bf1185835a8c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_201805032104276.png](https://upload-images.jianshu.io/upload_images/5363507-aee6445b7413f466.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_201805032104277.png](https://upload-images.jianshu.io/upload_images/5363507-c026fc5d44695074.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_201805032104278.png](https://upload-images.jianshu.io/upload_images/5363507-6bf5a1038f6e82ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_201805032104279.png](https://upload-images.jianshu.io/upload_images/5363507-56e12a56c983eb5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042710.png](https://upload-images.jianshu.io/upload_images/5363507-5b8afbe53a150096.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042711.png](https://upload-images.jianshu.io/upload_images/5363507-61763560ce2647a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042712.png](https://upload-images.jianshu.io/upload_images/5363507-5349e67b4137e147.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042713.png](https://upload-images.jianshu.io/upload_images/5363507-06237a29af38aa35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042715.png](https://upload-images.jianshu.io/upload_images/5363507-abbb3fccf5d9715b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042716.png](https://upload-images.jianshu.io/upload_images/5363507-090a65c5936c25a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042717.png](https://upload-images.jianshu.io/upload_images/5363507-b8d35b138339bd27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042718.png](https://upload-images.jianshu.io/upload_images/5363507-4d7710ea813724d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042719.png](https://upload-images.jianshu.io/upload_images/5363507-7c308b7b511cf4b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042720.png](https://upload-images.jianshu.io/upload_images/5363507-f8ca1641e8f4717a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042721.png](https://upload-images.jianshu.io/upload_images/5363507-326bbd34b0b27cf4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042722.png](https://upload-images.jianshu.io/upload_images/5363507-981e758b0a841c16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042723.png](https://upload-images.jianshu.io/upload_images/5363507-010de004372e61a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042724.png](https://upload-images.jianshu.io/upload_images/5363507-dc886a8e7f6407db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042725.png](https://upload-images.jianshu.io/upload_images/5363507-85a49ae448ad0622.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042726.png](https://upload-images.jianshu.io/upload_images/5363507-ada6b048ce31fe39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042728.png](https://upload-images.jianshu.io/upload_images/5363507-4435d020c74c07fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042729.png](https://upload-images.jianshu.io/upload_images/5363507-1ffddd8f8e5cdcd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042730.png](https://upload-images.jianshu.io/upload_images/5363507-67976b5e395d92df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042731.png](https://upload-images.jianshu.io/upload_images/5363507-7c0a5003bfbbca37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042732.png](https://upload-images.jianshu.io/upload_images/5363507-1270e8f3c511090d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042733.png](https://upload-images.jianshu.io/upload_images/5363507-f59a22fe820f2c30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042734.png](https://upload-images.jianshu.io/upload_images/5363507-1c997c8c024a162f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042735.png](https://upload-images.jianshu.io/upload_images/5363507-e1c1040785150ff5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042736.png](https://upload-images.jianshu.io/upload_images/5363507-fbc31b100749deef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042737.png](https://upload-images.jianshu.io/upload_images/5363507-bef2edf3bbe9164d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![微信图片_2018050321042738.png](https://upload-images.jianshu.io/upload_images/5363507-e033fffdbf267628.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  以上就是基本的用法和分析，谢谢