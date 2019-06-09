---
title: Vue.js学习No.5(WebPack配置二)
date: 2019-06-09 10:45:42
tags: WebPack
---

* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
* **学习的内容如下**

* **开始**

* **当我们需要频繁的修改main.js**
* 我们每次都需要去构建，这样显得很麻烦，`webpack .\src\main.js -o .\dist\bundle.js`，通过配置文件去配置每次的打包入口和出口
* 1、创建`webpack.config.js`
![image.png](https://upload-images.jianshu.io/upload_images/5363507-41a14b2df29e5dd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-ee502444e2cfa8d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 2、输入`webpack` 就可以直接打包
![image.png](https://upload-images.jianshu.io/upload_images/5363507-defb44e14b4dc359.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 3、生成相对应的文件
![image.png](https://upload-images.jianshu.io/upload_images/5363507-459a07ded92e6952.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **当我们在控制台直接输入` webpack`命令执行的时候，`webpack`做了几个步骤**
* 1、没有使用`webpack` 指定入口和出口
*  2、`webpack` 就会去项目中的根目录中，找一个`webpack.config.js `的配置文件
* 3、当找到了这个配置文件后，`webpack`就回去执行这个配置文件，解析执行完配置文件后，就得到了配置文件，导出配置对象
* 4、当`webpack`拿到配置对象了，就拿到了配置文件中入口和出口，然后进行打包和构建

* **使用webpack-dev-server工具实现自动的打包编译的功能**

* 1、运行`npm i webpack -dev -server -D`安装工具
![image.png](https://upload-images.jianshu.io/upload_images/5363507-80e35fdfd6da20fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/5363507-1c5769b5161cec3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 2、用法和webpack一样 

* 3、由于在本地安装的，所以无法使用脚本命令，只能安装到全局-g才能正常的使用
![image.png](https://upload-images.jianshu.io/upload_images/5363507-07f69b2e1c331764.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 4、脚本命令
在`pcakage.json`中添加`   "dev": "webpack-dev-server"`
![image.png](https://upload-images.jianshu.io/upload_images/5363507-9c6dbe7847431b9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 5、然后就可以在终端输入命令`npm run dev`

![image.png](https://upload-images.jianshu.io/upload_images/5363507-fb472af8c33d9e61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
h:\20190305\code\webpack>npm run dev

> yes@1.0.0 dev h:\20190305\code\webpack
> webpack-dev-server

i ｢wds｣: Project is running at http://localhost:8080/
i ｢wds｣: webpack output is served from /
i ｢wdm｣: Hash: 8b5966bb4652f34f42c7
Version: webpack 4.30.0
```
* 6、打开本地浏览器 ` http://localhost:8080/`，然后就可以看到,说明打包成功了

![image.png](https://upload-images.jianshu.io/upload_images/5363507-3406849aea16ffb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 7、注意点，当我们改变main.js中的内容，并且保存的时候，我们就可以自动的打包

* 8、webpack-dev-server帮我们打包生成的bundle.js文件，并没有存放到磁盘上，而是在电脑的内存中
![image.png](https://upload-images.jianshu.io/upload_images/5363507-e19b3345014aaa57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
打开根路径的bundle，打包成的服务器的地址，项目中找不到，但是可以自动打包 
```
      <!-- webpack-dev-server帮我们打包生成的bundle.js文件，并没有存放到磁盘上，而是在电脑的内存中 -->
        <script src="/bundle.js"></script>
```
* 9、可以认为这个webpack-dev-server工具，以一种虚拟的形式托管到了项目的根目录中，虽然看不到，但是可以认为和dist 平级的文件，为啥是这样做呢，原因就是快。**速度快**


* **webpack-dev-server工具常用的参数一**
*     配置端口号和自动打开浏览器："dev": "webpack-dev-server --open --port 3000"
* `--contentBase src` 可以自动的打开 index.html的文件，而不用去重新选择 src
![image.png](https://upload-images.jianshu.io/upload_images/5363507-207321bd2577edbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* ` --hot` 可以实现浏览器无刷新的重载，修改完样式，页面根本没有刷新，但是能够重新加载最新的代码
 ·`"dev": "webpack-dev-server --open --port 3001 --contentBase src --hot"`
* 完整的结果
![image.png](https://upload-images.jianshu.io/upload_images/5363507-74122e1777a1a94f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **webpack-dev-server工具常用的参数二**
* 这样配置和上面的效果一样
![image.png](https://upload-images.jianshu.io/upload_images/5363507-5e956e3a08d4802a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/5363507-697dc398a1c42fd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 最终的效果图
![image.png](https://upload-images.jianshu.io/upload_images/5363507-e7847782f56c0b90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
