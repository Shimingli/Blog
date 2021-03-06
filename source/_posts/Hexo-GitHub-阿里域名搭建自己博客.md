---
title: Hexo+GitHub+阿里域名搭建自己博客
date: 2018-08-14 18:39:00
tags: [Hexo,阿里域名,博客]
---

##  效果
* 博客的地址：[我的博客](http://www.shiming.site/)
<!--  more  -->
   * PC端的效果
![PC端的效果](https://upload-images.jianshu.io/upload_images/5363507-ba3860adcb584a57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   * 手机端效果
![手机端效果一](https://upload-images.jianshu.io/upload_images/5363507-ad4a55e07594b58a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![手机端效果二](https://upload-images.jianshu.io/upload_images/5363507-c5ae27aabcc81fa5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 最近抽空终于搭建完成，网上的资料很多，这篇文章只记录下遇到的问题，如果你想搭建一个博客！可能对你有些帮助。


## Hexo+GitHub
  * 现在网络上很多，有好多的文章写得很好，在这里推荐两篇文章！只要按照这步骤来，基本上都可以完成!没有什么技术的难度，就是体力活☺☺☺
    *  [基于Hexo+GitHub Pages 搭建博客详细教程](https://blog.csdn.net/u011974987/article/details/51331822/)
    *  [HEXO+Github,搭建属于自己的博客](https://www.jianshu.com/p/465830080ea9)
  
## 域名
  * 域名我是在阿里云上买的 [阿里云](https://wanwang.aliyun.com/?spm=5176.8142029.735711.56.a72376f4MMmf6X) 
  * 输入你想要买的域名地址，如图所示!选择一个您觉得合适的域名，加入到清单，然后购买就可以了
![域名](https://upload-images.jianshu.io/upload_images/5363507-bbd42440357260be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 这个是阿里云的代金券！[阿里云代金券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=ikk1hmxh)
* 购买成功了，进入域名控制台
![控制台](https://upload-images.jianshu.io/upload_images/5363507-b2dfbb9ab09bd1d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 点击解析设置，然后添加记录，就会出现下面的窗口
![设置](https://upload-images.jianshu.io/upload_images/5363507-a1df65f662543ba9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 设置值，结果如下
![设置结果](https://upload-images.jianshu.io/upload_images/5363507-55824b1a93001047.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  * 这个记录值是什么？如图所示，其实就是 ：`https://shimingli.github.io/` 的`ip`值，当然你需要设置成你的
  ![image.png](https://upload-images.jianshu.io/upload_images/5363507-eca8fae000cea2c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 设置完成了，还有关键的一步，就是实名认证，认证通过了，域名才可以使用。

## `GitHub Pages`的设置
  * 进入到 `name.github.io `的工程目录下，然后进入到`Settings`,最后定位到`Github Pages`的选项,输入你自己购买的域名，我这域名 7块一年，物超所值！
![settings](https://upload-images.jianshu.io/upload_images/5363507-87e99bbe6ab338b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 主题选择
*  [有哪些好看的Hexo主题](https://www.zhihu.com/question/24422335)
* 个人看了几个，我还是比较喜欢这个 [Yelee](https://github.com/MOxFIVE/hexo-theme-yelee)。感觉比较闷骚
*  `clone`到本地
```
git clone https://github.com/MOxFIVE/hexo-theme-yelee.git themes/yelee
```
* 使用的步骤建议参考 官方的文档，非常的详细 [Yelee 主题使用说明 [简中] ](http://moxfive.coding.me/yelee)
![目录结构](https://upload-images.jianshu.io/upload_images/5363507-749dd96e0302437f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   * `public`:`pull`到`Github`
   * `themes`：博客的主题`
    * ` soure`：就是文章存放的地方

* 然后把`clone`的`yelee`放到`themes`目录下

![image.png](https://upload-images.jianshu.io/upload_images/5363507-18d0bbbd77f7c421.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 在博客的根目录下设置`theme: Yelee`
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: Yelee
```
![设置](https://upload-images.jianshu.io/upload_images/5363507-c1433f718015480e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##  `Hexo` 几个常用的命令
```
hexo generate (hexo g) 生成静态文件，会在当前目录下生成一个新的叫做public的文件夹
hexo server (hexo s) 启动本地web服务，用于博客的预览
hexo deploy (hexo d) 部署博客到远端服务器
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
```
* 简写：
```
$ hexo n == hexo new
$ hexo g == hexo generate
$ hexo s == hexo server
$ hexo d == hexo deploy
```
* 如果一切配置ok的话，打开[http://localhost:4000/](http://localhost:4000/) 就可以看到刚才搭建的本地博客了，Hexo会默认生成一个Hello World的博文。

## 遇到的问题
*  `Hexo`中的`Yelee`主题，首页不显示文章,需要把`themes`中的文件`_config.yml`的`search`中的`on: true`的注释去掉，同时必须注释掉`onload`，同时需要去安装插件!,然后就可以检索了。
```
# Local Site Search | 本地站内搜索
## Insatall below plugin to take effect | 使用搜索需先安装对应插件
## https://github.com/PaicHyperionDev/hexo-generator-search
search: 
  on: true
  #onload: false     （Hexo中的Yelee主题，首页不显示文章）
  ## true: get search.xml file when the page has loaded
  ## false: get the file when search box gets focus
```

#### 再来一个好玩的，如何统计网页的访问量？如下图所示，请忽略我的可怜的访问量？

![访问量](https://upload-images.jianshu.io/upload_images/5363507-4af81e030d02f92c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-683a8f05531721ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 在`yelee` 主题下的 `_config.yml`文件中，有个百度统计
```

#   百度统计
# 百度统计 http://sitecenter.baidu.com/sc-web/
# 查看代码，填入 //hm.baidu.com/hm.js? 之后的内容     
baidu_tongji: 
```
* 登录注册后填写网站,然后获取代码

![百度统计.png](https://upload-images.jianshu.io/upload_images/5363507-6092eaba96e7c1d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![获取代码的详情](https://upload-images.jianshu.io/upload_images/5363507-b3cce935c7a79442.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 这是我的获取代码的详情
```
<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?1add6c64d9c467153aaed18d249d81c4";
  var s = document.getElementsByTagName("script")[0]; 
```
* 把这一串东西 `1add6c64d9c467153aaed18d249d81c4`放到百度统计一栏，即可。
![百度统计](https://upload-images.jianshu.io/upload_images/5363507-865d469abea10fea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  然后 `hexo g` `hexo s` `hexo d` ,然后`push`到 `GitHub`.他会自动的检测代码。
