---
title: Hement：MVP架构设计（一）
date: 2018-12-04 09:20:41
tags: [MVP,架构]
---
* Hement  ： 取英文前字母，意思为快乐享受，希望我们在敲代码的工程中是快乐享受的意思

![image.png](https://upload-images.jianshu.io/upload_images/5363507-7f23cbaa07e2f6cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 16年的时候我写过一篇博客基于`Retorfit+Rxjava+Rxandroid`的网络架构[MVP网络框架(Retorfit+Rxjava+Rxandroid)](https://www.jianshu.com/p/141ee58eb143)，使用的是`Rxjava1.0`，其中的`Model` 层搭建过于简单，加重了`Presenter`的任务，同时也没有很好的解耦，没有使用`Dagger2`依赖注入框架,`Presenter`的生命周期没有和`Acivity`绑定等等问题。故有此文章

<!--  more  --> 
#### 1、什么是MVP架构？
* **Model-view-presenter**，简称MVP，是计算机软件设计工程中一种对针对MVC模式，再审议后所延伸提出的一种软件设计模式。

     * **MVC模式**（Model–view–controller）是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）。
      * MVC模式最早由`Trygve Reenskaug`在1978年提出，是施乐帕罗奥多研究中心在20世纪80年代为程序语言`Smalltalk`发明的一种软件架构。**MVC模式**的目的是实现一种动态的程序设计，使后续对程序的修改和扩展简化，并且使程序某一部分的重复利用成为可能。除此之外，此模式通过对复杂度的简化，使程序结构更加直观。软件系统通过对自身基本部分分离的同时也赋予了各个基本部分应有的功能。专业人员可以通过自身的专长分组：
          *   控制器（Controller）- 负责转发请求，对请求进行处理。
          *   视图（View） - 界面设计人员进行图形界面设计。
          *   模型（Model） - 程序员编写程序应有的功能（实现算法等等）、数据库专家进行数据管理和数据库设计(可以实现具体的功能)。

*  **Model-view-presenter (MVP)** 是用户界面设计模式的一种，被广范用于便捷自动化单元测试和在呈现逻辑中改良分离关注点（separation of concerns）。

     *   **Model** 定义用户界面所需要被显示的数据模型，一个模型包含着相关的业务逻辑。
     *   **View** 视图为呈现用户界面的终端，用以表现来自 Model 的数据，和用户命令路由再经过 Presenter 对事件处理后的数据。
     *   **Presenter** 包含着组件的事件处理，负责检索 Model 获取数据，和将获取的数据经过格式转换与 View 进行沟通。

![MVP模式.jpg](https://upload-images.jianshu.io/upload_images/5363507-6bc56d411705b093.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2、Hement中的MVP架构
* View(UI层)：这是`Activities`、`Fragments`和其他标准`Android components `组件所在的地方。它负责向用户显示从演示者接收的数据。它还处理用户交互和输入（点击事件等），并在需要时触发Presenter中的正确操作。
* Presenter：presenters订阅“DataManager”提供的RxJava可观察对象。他们负责处理订阅生命周期，分析/修改“DataManager”返回的数据，并调用视图中的适当方法来显示数据。

* Model (Data Layer)：它负责检索、保存、缓存和消息数据。它可以与本地数据库和其他数据存储以及`restful APIs `或第三方`SDK`通信。它分为两部分：`helpers` 和一个`DataManager`。`helpers` 的数量因项目而异，每个`helper`都具有非常特定的功能，例如，与`API`对话或在“SharedPreferences”中保存数据。`DataManager`使用`RxJava`操作符组合并转换来自不同`helper` 的输出，因此它可以：1）向演示者提供有意义的数据；2）总是一起发生的组动作。该层还包含定义数据结构如何的实际模型类。

![Android中的MVP模式.jpg](https://upload-images.jianshu.io/upload_images/5363507-cc2c20e2e4937c87.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 从右到左看图表：

  *  Helpers(Model)：一组类，每个类都有非常特定的责任。它们的功能可以从与`API`或数据库对话到实现一些特定的业务逻辑。每个项目都有不同的`Helper`，但最常见的是：
  *  DataBaseHelper：它处理从本地`SQLite`数据库中插入、更新和检索数据。它的方法返回发出普通`Java`对象的`Rx Observables`可观测值。
   * PreferencesHelper：它优先保存和获取来自`SharedPreferences`的数据，它可以直接返回`Observables`或纯`Java`对象。
   *  Retrofit services：`Retrofit`与`Restful APIs`进行通信的接口，每个不同的`API`都有自己的`Retrofit`服务。它们返回`Rx Observables`。
   * Data Manager (Model)：它是体系结构的一个关键部分。它保留对每个`Helpers`类的引用，并使用它们来满足来自`presenters`的请求。
   *  Presenters：订阅`DataManager`提供的可观察对象，并处理数据，以便在视图中调用正确的方法。
   *  Activities,Fragments,ViewGroups(View)：实现演示者可以调用的一组方法的标准Android组件。它们还处理用户交互，例如单击并相应地通过调用`Presenter`中的适当方法进行操作。这些组件还实现与框架相关的任务，比如管理Android生命周期、填充视图等。
    * Event Bus：它允许向视图组件通知模型中发生的某些类型的事件。通常，‘DataManager’发布事件，然后这些事件可以被活动和片段订阅。事件总线用于与仅一个屏幕无关并且具有广播性质的非常特定的操作例如，用户已经退出登录.

* 未完待续。。。。。

* GitHub地址：[Hement](https://github.com/Shimingli/Hement),持续更新中

* 最后说明几点
  * tks wiki [MVC](https://zh.wikipedia.org/wiki/MVC) 
  * 我写文章很少说自己的观点，这是第一次表明自己的观点，项目本身不大的情况，也没有达到某种量级的时候，不要使用组件化和插件化，构建时间和找问题的时间太多了，反而浪费开发的时间，自我感觉，组件化就是大厂玩的东西，我不是说不好，而是对于我们大多数的公司，是完全用不上组件化的，及时用的上，搞明白轮子的原因都要花费好多的时间和经历，但是自己的项目一定要清晰，至少自己使用起来不那么的讨厌！
  * `Event Bus` 但是这个事件总线有一定局限性，如果这个不存在生命周期的话，是不能够接受到的这个事件。例如：你不能发一个事件，让某个没有打开的View调动通知栏。
  * tks [Model-view-presenter](https://zh.wikipedia.org/wiki/Model-view-presenter)
  * **[android-boilerplate](https://github.com/Shimingli/android-boilerplate)**
