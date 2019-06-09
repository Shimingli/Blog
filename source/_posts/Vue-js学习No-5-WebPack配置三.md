---
title: Vue.js学习No.5(WebPack配置三)
date: 2019-06-09 10:47:31
tags: WebPack
---
* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
* **学习的内容如下**

* **开始**

#### npm介绍

* 说明：npm（node package manager）是nodejs的包管理器，用于node插件管理（包括安装、卸载、管理依赖等） 

* cnpm跟npm用法完全一致，只是在执行命令时将npm改为cnpm。

####  html-webpack-plugin插件
* 安装` npm i html-webpack-plugin`插件
* 生成在内存中的html实例
![image.png](https://upload-images.jianshu.io/upload_images/5363507-85a5322bc1d1455d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 1、内存中生成HTML页面插件，只要是插件，就比放到 plugins中去
const htmlplugin=require('html-webpack-plugin')
* 2、创建在内存中生成html页面的插件
```
  plugins:[
        //创建在内存中生成html页面的插件
        new  htmlplugin({
            //指定模板页面，根据指定的页面路径，去生成内存中的页面
         template:path.join(__dirname,"./src/index.html"),
         //指定生成的页面的名称
         filename:"index.html"

        })
    ]
```
* 多生成了一个这个东西
![image.png](https://upload-images.jianshu.io/upload_images/5363507-4ff24090c433f32b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 当我们使用了 html-webpack-plugin 之后，我们不再需要手动处理 bundle.js,因为插件能帮我们应用 
![image.png](https://upload-images.jianshu.io/upload_images/5363507-6fad8d84dfa573b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*  html-webpack-plugin作用
    *  1、自动在内存中根据指定的页面生成一个内存的页面
     *  2、自动把打包好的bundle.js 追加到页面中去

####  如何加载自己项目中的css文件
#####  方法一：   导入自己的css  有点延迟加载 css会发生二次请求，不推荐这么导入 ` <link rel="stylesheet" href="./css/index.css"> `
#####  方法二： 
* 1、在main.js文件中导入`import "./css/index.css"`
![image.png](https://upload-images.jianshu.io/upload_images/5363507-6ddd47168069f4c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 2、运行起来但是会报错，因为webpack只能打包js类型的文件，无法处理其他非JS的文件
![image.png](https://upload-images.jianshu.io/upload_images/5363507-0e29d42dc04660bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 3、安装`npm i style-loader css-loader -D`
* 4、打开 webpack.config.js 这个配置文件，新建module节点，用于配置所有第三方模块的加载器
![image.png](https://upload-images.jianshu.io/upload_images/5363507-a9a2b33b06edbd4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####  webpack打包过程
* 在打包过程中，先校验文件，是js文件就直接打包，不是的话，先拿到后缀名，然后在 webpack.config.js中找对应的匹配规则，如果找不到就会报错，就好像上面的图一样
* 调用的顺序，从右到左的顺序，从后往前调用的
![image.png](https://upload-images.jianshu.io/upload_images/5363507-932640bb98d71735.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 当最后的一个loader调用完毕，会把处理的结果，直接交给webpack打包合并，最后输出到 bundle.js中去

####  less文件的导入
* 1、安装npm i less-loader -D
![image.png](https://upload-images.jianshu.io/upload_images/5363507-f28f9463a40a09a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 2、缺少内部驱动 安装npm i less
* 3、配置处理less文件的loader的规则`  test:/\.less$/,use:["style-loader","css-loader","less-loader"]`

![image.png](https://upload-images.jianshu.io/upload_images/5363507-3d8ac9bcbc34c209.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####  scss文件的导入

* 1、安装`npm i sass-loader -D`
* 2、安装内部依赖` npm i node-sass -D `  最好使用 `cnpm i node-sass-D `来安装
![image.png](https://upload-images.jianshu.io/upload_images/5363507-0649288ba9c0fd97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

