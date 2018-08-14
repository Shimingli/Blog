---
title: Go语言使用MySQL数据库
date: 2018-07-06 16:03:55
tags: [Go,MySQL]
---

* 本文主要介绍使用Go语言使用MySQL的搭建工作，适合移动端同学自己学习Go语言玩玩
* 前段时间撸了一个简单Spring-boot的Demo，最近学习了Go语言才发现，Go做起来更加的简单，我不推崇，不崇拜，但是我还得说一句，Go语言真好！！！
<!--  more  -->
#### 一、搭建Go语言的环境
* 安装Go的时候看到需要设置GOPATH变量，Go从1.1版本到1.7必须设置这个变量，而且不能和Go的安装目录一样，这个目录用来存放Go源码，Go的可运行文件，以及相应的编译之后的包文件。所以这个目录下面有三个子目录：`src、bin、pkg`
* 从go 1.8开始，GOPATH环境变量现在有一个默认值，如果它没有被设置。在Windows上默认为%USERPROFILE%/go。
* 我自己设置的环境变量
![我自己设置的环境变量.png](https://upload-images.jianshu.io/upload_images/5363507-1a32ce15cec746e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 安装完成大概是这个效果
![image.png](https://upload-images.jianshu.io/upload_images/5363507-bf714c06911e1515.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* GOPATH 目录约定有三个子目录：
    * src 存放源代码（比如：.go .c .h .s等）
    * pkg 编译后生成的文件（比如：.a）
    * bin 编译后生成的可执行文件
#### 二、Go开发工具
* 由于Go开发的工具巨多，作为一个菜鸟，目前不知道利弊，就当总结下资料，同时我自己使用的工具是`IDEA`，当然不是免费版，去网上搞个秘钥直接破解，因为`IDEA`真的很强大，这就不多提了，建议如果还在使用的`Android Studio`写安卓程序的小伙伴，可以试试使用`IDEA`，都差不多

* 1、LiteIDE
 LiteIDE是一款专门为Go语言开发的跨平台轻量级集成开发环境（IDE），由visualfc编写。
     *    LiteIDE安装
     *    下载地址 [http://sourceforge.net/projects/liteide/files](http://sourceforge.net/projects/liteide/files)
     *    源码地址 [https://github.com/visualfc/liteide](https://github.com/visualfc/liteide)
* 2、Sublime Text 
   参考这个博客地址 http://blog.jobbole.com/88648/
* 3、Visual Studio Code
    vscode是微软基于Electron和web技术构建的开源编辑器, 是一款很强大的编辑器。开源地址:[https://github.com/Microsoft/vscode](https://github.com/Microsoft/vscode)
* 4、Eclipse
Eclipse！但是我好久都没用了，哈哈！！！ 
* 5 、IntelliJ IDEA
idea是通过一个插件来支持go语言的高亮语法,代码提示和重构实现。虽然开发Go语言社区免费版就可以了，但是还是建议搞个正式版（具体百度，嘿嘿），如果不行就使用社区免费版！我自己就是使用的`IDEA`
具体的步骤如下，就好像install个插件！
![image.png](https://upload-images.jianshu.io/upload_images/5363507-1f9bd04303496354.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 三、搭建数据库，mysql + navicat
* 1 安装MySQL
我的安装包是公司的Java工程师发给我的，官网下载比较慢，文件为mysql-5.7.20-windx64.msi
![image.png](https://upload-images.jianshu.io/upload_images/5363507-42c90bc4a709bb52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击直接安装，在第二步Choosing a Setup Type的时候，选择Server only
![image.png](https://upload-images.jianshu.io/upload_images/5363507-31b6046bd9445715.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后一直Next，在输入数据库密码的时候输入App123，输入用户名的时候需要输入root，同时要把这两个记下来，后面连接数据库的时候需要用到 

![image.png](https://upload-images.jianshu.io/upload_images/5363507-9fd5249356e29445.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*  2、用navicat操作数据库
navicat是个工具，可以直接到百度下载，然后连接数据库，输入你的密码和用户名即可
![image.png](https://upload-images.jianshu.io/upload_images/5363507-8ac63c8168ab91df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
新建一个数据库godbdemo，为什么叫godbdemo，因为在Go里面等下需要配置的数据库就叫它。
![image.png](https://upload-images.jianshu.io/upload_images/5363507-e2a3340262a3a145.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 3、创建表userinfo
```
CREATE TABLE `userinfo` (
	`uid` INT(10) NOT NULL AUTO_INCREMENT,
	`username` VARCHAR(64) NULL DEFAULT NULL,
	`department` VARCHAR(64) NULL DEFAULT NULL,
	`created` DATE NULL DEFAULT NULL,
	PRIMARY KEY (`uid`)
);
```
使用Navicat Premium 创建数据库
![使用Navicat Premium 创建数据库1.png](https://upload-images.jianshu.io/upload_images/5363507-365e288d8136449a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![使用Navicat Premium 创建数据库2.png](https://upload-images.jianshu.io/upload_images/5363507-d8bd1e8c611c3002.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![使用Navicat Premium 创建数据库3.png](https://upload-images.jianshu.io/upload_images/5363507-0d98a00f4b6f9475.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* ok，准备工作差不多了
#### 四、GitHub依赖的管理
* 不可避免的我们会使用外部的依赖包包。go在这方面做的非常飘逸。go没有像java使用maven来管理依赖包、包版本，而是直接使用GOPATH来管理外部依赖。
* Go官方没有提供数据库驱动，而是为开发数据库驱动定义了一些标准接口，开发者可以根据定义的接口来开发相应的数据库驱动，这样做有一个好处，只要是按照标准接口开发的代码， 以后需要迁移数据库时，不需要任何修改
* 1、MySQL驱动
 我自己使用的是这个驱动
 [https://github.com/go-sql-driver/mysql](https://github.com/go-sql-driver/mysql) 支持database/sql，全部采用go写。
* 主要理由：
     *   这个驱动比较新，维护的比较好
     *   完全支持database/sql接口
     *   支持keepalive，保持长连接,虽然[星星](http://www.mikespook.com/)fork的mymysql也支持keepalive，但不是线程安全的，这个从底层就支持了keepalive。
* 2、使用`git`拉取代码
```
$ go get -u github.com/go-sql-driver/mysql
```
* 最终的效果如下
![goPATh目录下的github目录的src目录.png](https://upload-images.jianshu.io/upload_images/5363507-8260346448e2e69b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![goPATh目录下的github目录的pkg目录.png](https://upload-images.jianshu.io/upload_images/5363507-2dca9092efa06cb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 5、代码实现 

```
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	//其实这个就是Go设计的巧妙之处，我们在变量赋值的时候经常看到这个符号，它是用来忽略变量赋值的占位符，那么包引入用到这个符号也是相似的作用，这儿使用_的意思是引入后面的包名而不直接使用这个包中定义的函数，变量等资源。
)

func init() {
//	Go中支持MySQL的驱动目前比较多，有如下几种，有些是支持database/sql标准，而有些是采用了自己的实现接口,常用的有如下几种:
//
//https://github.com/go-sql-driver/mysql 支持database/sql，全部采用go写。
//https://github.com/ziutek/mymysql 支持database/sql，也支持自定义的接口，全部采用go写。
//https://github.com/Philio/GoMySQL 不支持database/sql，自定义接口，全部采用go写。
//	接下来的例子我主要以第一个驱动为例(我目前项目中也是采用它来驱动)，也推荐大家采用它，主要理由：
//
//	这个驱动比较新，维护的比较好
//	完全支持database/sql接口
//	支持keepalive，保持长连接,虽然星星fork的mymysql也支持keepalive，但不是线程安全的，这个从底层就支持了keepalive。
}
func main() {
//	user@unix(/path/to/socket)/dbname?charset=utf8
//user:password@tcp(localhost:5555)/dbname?charset=utf8
//user:password@/dbname
//user:password@tcp([de:ad:be:ef::ca:fe]:80)/dbname


	//spring.datasource.url =jdbc:mysql://localhost:3306/test
	//spring.datasource.username = root
	//#  注意密码的问题 ，数据的密码的
	//spring.datasource.password = App123
    // sql.Open()函数用来打开一个注册过的数据库驱动，go-sql-driver中注册了mysql这个数据库驱动，第二个参数是DSN(Data Source Name)，它是go-sql-driver定义的一些数据库链接和配置信息   如上的图片

	db, err := sql.Open("mysql", "root:App123@tcp(localhost:3306)/godbdemo?charset=utf8")
	checkErr(err)

	//插入数据 db.Prepare()函数用来返回准备要执行的sql操作，然后返回准备完毕的执行状态。
	stmt, err := db.Prepare("INSERT userinfo SET username=?,department=?,created=?")
	checkErr(err)

	for i:=1;i<100000 ;i++  {
		//stmt.Exec()函数用来执行stmt准备好的SQL语句
		res, err := stmt.Exec("shiming", "研发部门", "20180706")
		checkErr(err)
		//shiming i== 9727 res=== {0xc0420a0000 0xc0422f5000}
		fmt.Println("shiming i==",i,"res===",res)

	}
	res, err := stmt.Exec("shiming", "研发部门", "20180706")
	checkErr(err)

	id, err := res.LastInsertId()
	checkErr(err)

	fmt.Println(id)
	//更新数据  传入的参数都是=?对应的数据，这样做的方式可以一定程度上防止SQL注入
	stmt, err = db.Prepare("update userinfo set username=? where uid=?")
	checkErr(err)

	res, err = stmt.Exec("shimingupdate", id)
	checkErr(err)

	affect, err := res.RowsAffected()
	checkErr(err)

	fmt.Println(affect)

	//查询数据  db.Query()函数用来直接执行Sql返回Rows结果。
	rows, err := db.Query("SELECT * FROM userinfo")
	checkErr(err)

	for rows.Next() {
		var uid int
		var username string
		var department string
		var created string
		err = rows.Scan(&uid, &username, &department, &created)
		checkErr(err)
		fmt.Println(uid)
		fmt.Println(username)
		fmt.Println(department)
		fmt.Println(created)
	}

	//删除数据
	//stmt, err = db.Prepare("delete from userinfo where uid=?")
	//checkErr(err)
	//
	//res, err = stmt.Exec(id)
	//checkErr(err)
	//
	//affect, err = res.RowsAffected()
	//checkErr(err)
	//
	//fmt.Println(affect)

	db.Close()

}

func checkErr(err error) {
	if err != nil {
		panic(err) //大概相当于  Java中的异常
	}
}

```

* `sql.Open()`函数用来打开一个注册过的数据库驱动，`go-sql-driver`中注册了`mysql`这个数据库驱动，第二个参数是`DSN(Data Source Name)`，它是`go-sql-driver`定义的一些数据库链接和配置信息。它支持如下格式：
```
user@unix(/path/to/socket)/dbname?charset=utf8
user:password@tcp(localhost:5555)/dbname?charset=utf8
user:password@/dbname
user:password@tcp([de:ad:be:ef::ca:fe]:80)/dbname
```
在我的代码中是使用的是这种的方式`user:password@tcp(localhost:5555)/dbname?charset=utf8`,也就是这种的效果 `db, err := sql.Open("mysql", "root:App123@tcp(localhost:3306)/godbdemo?charset=utf8")` 。
* 其中的   `root`是数据库的用户名，  `App123`是数据库的密码。
* 几个关键的方法
  * db.Prepare()函数用来返回准备要执行的sql操作，然后返回准备完毕的执行状态。
  * db.Query()函数用来直接执行Sql返回Rows结果。
   *  stmt.Exec()函数用来执行stmt准备好的SQL语句
#### 6、最终的效果如下
由于我的代码中有个循环
```
	for i:=1;i<100000 ;i++  {
		//stmt.Exec()函数用来执行stmt准备好的SQL语句
		res, err := stmt.Exec("shiming", "研发部门", "20180706")
		checkErr(err)
		//shiming i== 9727 res=== {0xc0420a0000 0xc0422f5000}
		fmt.Println("shiming i==",i,"res===",res)

	}
```
* 点击运行

![image.png](https://upload-images.jianshu.io/upload_images/5363507-4f27432bdb554d6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 最后，说明一点：以上的[Demo](https://github.com/Shimingli/GoDemo)是自己搭建玩玩，一定会有些差错，请大牛不要见怪哈~~~
