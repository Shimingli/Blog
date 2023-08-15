# Blog

* 新電腦安裝東西  https://blog.csdn.net/smallnetter/article/details/98492603


我的博客



* 使用`yelee`主题
* 一共三十一篇文章
##  `Hexo` 几个常用的命令
```
hexo generate (hexo g) 生成静态文件，会在当前目录下生成一个新的叫做public的文件夹
hexo server (hexo s) 启动本地web服务，用于博客的预览
hexo deploy (hexo d) 部署博客到远端服务器
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
npm i hexo-theme-matery
```
* 简写：
```
$ hexo n == hexo new
$ hexo g == hexo generate
$ hexo s == hexo server
$ hexo d == hexo deploy


```
* **npm uninstall -g  hexo-prism-plugin** 卸载某个插件 
* hexo s 发生了错误的时候：https://segmentfault.com/q/1010000008638413

* 如果报错
```
Template render error: (unknown path)
  Error: filter not found: nameope
    at Object._prettifyError (E:\blog\hexo\bloghexo\node_modules\nunjucks\src\li
b.js:36:11)
    at E:\blog\hexo\bloghexo\node_modules\nunjucks\src\environment.js:545:19
    at Template.root [as rootRenderFunc] (eval at _compile (E:\blog\hexo\bloghex
o\node_modules\nunjucks\src\environment.js:615:18), <anonymous>:27:3)
    at Template.render (E:\blog\hexo\bloghexo\node_modules\nunjucks\src\environm
ent.js:538:10)
    at Environment.renderString (E:\blog\hexo\bloghexo\node_modules\nunjucks\src
\environment.js:362:17)
    at Promise.fromCallback.cb (E:\blog\hexo\bloghexo\node_modules\hexo\lib\exte
nd\tag.js:62:48)
    at tryCatcher (E:\blog\hexo\bloghexo\node_modules\bluebird\js\release\util.j
s:16:23)
    at Function.Promise.fromNode.Promise.fromCallback (E:\blog\hexo\bloghexo\nod
e_modules\bluebird\js\release\promise.js:180:30)
    at Tag.render (E:\blog\hexo\bloghexo\node_modules\hexo\lib\extend\tag.js:62:
18)
    at Object.onRenderEnd (E:\blog\hexo\bloghexo\node_modules\hexo\lib\hexo\post
.js:282:20)
    at Promise.then.then.result (E:\blog\hexo\bloghexo\node_modules\hexo\lib\hex
o\render.js:65:19)
    at tryCatcher (E:\blog\hexo\bloghexo\node_modules\bluebird\js\release\util.j
s:16:23)
    at Promise._settlePromiseFromHandler (E:\blog\hexo\bloghexo\node_modules\blu
ebird\js\release\promise.js:512:31)
    at Promise._settlePromise (E:\blog\hexo\bloghexo\node_modules\bluebird\js\re
lease\promise.js:569:18)
    at Promise._settlePromise0 (E:\blog\hexo\bloghexo\node_modules\bluebird\js\r
elease\promise.js:614:10)
    at Promise._settlePromises (E:\blog\hexo\bloghexo\node_modules\bluebird\js\r
elease\promise.js:693:18)
    at Async._drainQueue (E:\blog\hexo\bloghexo\node_modules\bluebird\js\release
\async.js:133:16)
    at Async._drainQueues (E:\blog\hexo\bloghexo\node_modules\bluebird\js\releas
e\async.js:143:10)
    at Immediate.Async.drainQueues (E:\blog\hexo\bloghexo\node_modules\bluebird\
js\release\async.js:17:14)
    at runCallback (timers.js:705:18)
    at tryOnImmediate (timers.js:676:5)
    at processImmediate (timers.js:658:5)

```

* 解决的思路是 ：[hexo/issues/2384](https://github.com/hexojs/hexo/issues/2384)
* 导致的原因是：不能使用 两个连起来的大括号，所以本文都是去掉的，但是呢，能在代码块中使用
```
 这种在正文使用会报错，代码块中使用不会
{{ ddd }}   

```

 
* tags 后面必须要有一个空格，要不然会编译抱错 
```

---
title: Vue.js学习No.5
date: 2019-04-24 11:16:29
tags: WebPack
---
```