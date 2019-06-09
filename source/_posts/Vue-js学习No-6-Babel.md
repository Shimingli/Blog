---
title: Vue.js学习No.6(Babel)
date: 2019-06-09 10:50:29
tags: Babel 
---
* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
* **学习的内容如下**

* **开始**
* 写在前面的话，最好使用cnpm 来安装包，要不然一堆错误

#### class是 ES6提供的新的语法，用来实现ES6中面向对象编程
```
//class是 ES6提供的新的语法，用来实现ES6中面向对象编程
class P{
    static info={name:"25",age:20}
}
//和 java 实现面向对象的方式完全一样 
var p=new P()
```
但是报错了
![image.png](https://upload-images.jianshu.io/upload_images/5363507-482030184570132e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
Module parse failed: Unexpected token (43:15)
You may need an appropriate loader to handle this file type.
| //class是 ES6提供的新的语法，用来实现ES6中面向对象编程
| class P{
>     static info={name:"25",age:20}
| }
| //和 java 实现面向对象的方式完全一样 
```
*  在webpack中，默认只能处理一部分ES6的新的语法。一些语法是处理不了的！需要借助第三放的loader把高级语法转为低级的语法


#### 通过Babel,帮我们将高级的语法转化为低级的语法 

##### 1、在webpack中，可以运行如下两套的命令，安装两套包去安装 Babel ,必须注意的是，字母的拼写` cnpm i babel-core babel-loader babel-plugin-transform-runtime -D`

![image.png](https://upload-images.jianshu.io/upload_images/5363507-0e3a01c3d992f5e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2、`cnpm i babel-preset-env babel-preset-stage-0 -D`
![image.png](https://upload-images.jianshu.io/upload_images/5363507-e7fe27935bcf825a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 3、打开webpack 的配置文件，在module节点下的rules数组中，添加一个新的匹配规则
` {test:/\.js$/,use:"bable-loader",exclude:/node_modules/}`

* exclude:/node_modules/
* 在配置babel的loader规则时候，必须把node_modules 目录，通过exclude 排除掉
* 因为：1、不排除node_modules拍除外的话，会把node_modules中所有的第三方的JS文件都打包编译，这样，会非常的消耗Cpu，打包的速度会非常的慢   !     2、如果Babel把node—moduls都打包完成了，项目也运行不起来

![image.png](https://upload-images.jianshu.io/upload_images/5363507-b9e4f8de4bad257f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 4、在项目的根目录中，新建一个叫做 .babelrc 的Babel的配置文件，这个配置文件属于JSON个数，所以在写 .babelrc 配置的时候，必须符合JSON语法规范，不能写注释，字符串必须用双引号

![image.png](https://upload-images.jianshu.io/upload_images/5363507-97e8e6ce82b9eb70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-4321c25ec56fffeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-6567b30aaca65d90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[Babel 是一个 JavaScript 编译器](https://www.babeljs.cn/)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-db54204887779151.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 最后就可以使用ES6语法特性