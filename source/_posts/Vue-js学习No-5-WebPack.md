---
title: Vue.js学习No.5
date: 2019-04-24 11:16:29
tags: WebPack
---
* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **学习的内容如下**
<!--  more  -->

* **开始**

* **npm的使用**
`npm `安装一些包会出现问题，很可能问题的原因是我们的网络。`npm` 的包的安装源有挺多，默认是`npm`，如果在国内，我们可以把安装源切换成` taobao `，这样安装的速度会快很多。
```
# 先用 npm 安装 nrm 小工具
npm install nrm --global

# 安装后查看现在的 npm 的安装源
nrm ls

# 切换使用 taobao 作为 npm 的安装源
nrm use taobao

# 查看当前安装源用的是什么
nrm ls
```
注意有时候你使用` taobao` 作为安装源也可能会遇到一些问题，这时候可以暂时再把安装源再切换成原来的 `npm` 。

![image.png](https://upload-images.jianshu.io/upload_images/5363507-d41175a240a8d748.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### webpack
* **网页中静态资源**
* JS 
  * .js  .jsx  .coffee .ts(TypeScript  C#)

*  CSS
   *  .css  .less .sass(基本上没有人用了)  .scss
*   Images 
    * .jpe   .png  .gif  .bmp  .svg 
*  字体文件
    * .svg .ttf   .eot  .woff  .woff2
*  （Fonts） 模板文件
     * .ejs   .jade  .vue(在weppack中定义组件的方式，推荐使用)

* **网页中静态资源多了的问题？**
* 1、加载速度慢，因为我们要发起很多的二次请求；
* 2、要处理赋复杂的依赖关系
* **如何解决问题？**
* 1、合并、压缩、精灵图、图片的Base64编码（url直接指向Base64）
* 2、使用requireJS、也可以使用webpack可以解决依赖关系
* **什么是webpack？**
* webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。基于Node.js 开发出来的前端工具

[webpack](https://www.webpackjs.com/concepts/)

  ![image.png](https://upload-images.jianshu.io/upload_images/5363507-c2469a49ed3553e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **完美的解决方法**
* 1、基于Gulp，基于 task 任务的
* 2、使用webpack，是基于整个项目进行构建的
   *  webpack自动化构建工具，可以完美的实现资源的合并、打包、压缩、混淆等诸多功能 

* **webpack的案例**
* jQuery 是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（或JavaScript框架）。jQuery设计的宗旨是“write Less，Do More”，即倡导写更少的代码，做更多的事情。它封装JavaScript常用的功能代码，提供一种简便的JavaScript设计模式，优化HTML文档操作、事件处理、动画设计和Ajax交互。
* 装jquery 
1、npm init
![image.png](https://upload-images.jianshu.io/upload_images/5363507-4f468bbe5c5799c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2、 npm i jquery -s
![image.png](https://upload-images.jianshu.io/upload_images/5363507-e41944ce279cccbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3、 自动生成
![image.png](https://upload-images.jianshu.io/upload_images/5363507-d084a8a4bc84a09d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* 由于ES6的代码，太高级了，浏览器解析不了，所以，这一行执行会报错`import $ from 'jquery'`
![image.png](https://upload-images.jianshu.io/upload_images/5363507-3822e356f4994bf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 如何解决呢？使用`webpack`帮助我们处理。
1、安装webpack `npm i webpack -g`
 `npm i webpack -g` 全局安装webpack，这样能够全局使用webpack

![image.png](https://upload-images.jianshu.io/upload_images/5363507-e375ed280c41e10f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、在项目的根目录中运行 `npm i webpack --save -dev` 安装到项目的依赖中去

* **使用webpack打包构建**
1、运行     `npm init` 初始化项目，使用npm管理项目中的依赖包
2、创建项目的目录结构
![image.png](https://upload-images.jianshu.io/upload_images/5363507-c2782ed1a656db32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3、使用  `npm i jquery --save` 安装jQuery的依赖包
4、创建    `main,js`并书写代码逻辑
5、` webpack .\src\main.js -o .\dist\bundle.js `最新的打包命令,意思是 ： webpack 入口文件路径 输出文件的路径
![image.png](https://upload-images.jianshu.io/upload_images/5363507-a1c43bc90e86e415.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* **打包完成**
![image.png](https://upload-images.jianshu.io/upload_images/5363507-6503bee61c3e0cb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


####  `webpack` 能做什么事情
*  1 、webpack 能够处理 JS 文件的互相依赖关系：理想状态一个项目只有一个main.js 但是不推荐
*  2、webpack 能够处理Js的兼容的问题，把高级的浏览器不识别的语法转化为低级的浏览器能够执行的
* **运行的命令格式 ：webpack  要打包的文件的路径  -o  输出文件的路径**