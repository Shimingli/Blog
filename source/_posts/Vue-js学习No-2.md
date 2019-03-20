---
title: Vue.js学习No.2
date: 2019-03-20 09:37:26
tags: Vue.js
---

* **欢迎关注我的公众号**

![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* **写在前面的话，非常的重要**
<!--  more  -->

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

* **学习的内容如下**



* **开始**
* 过滤器的定义方法 主要去解决时间显示的问题  过滤器调用的格式 ` {name | nameope }`

```
// Vue.filter("过滤器的名称",function(data){
        //      return data+"123"
        // })

  Vue.filter("msgFormat",function(msg,args){
             return msg+"arg"+args
        })
     <td>{item.Ctime | msgFormat('你好仕明同学')}</td>
```
<!--  more  -->

* 这个是干嘛的？？？ ` <link rel="stylesheet" href="./lib/bootstrap.min.css">`

* `@keyup.enter="add" ` 点击了回车键的话，就触发事件  全部的按键别名`  .enter  .tab .delete  .esc  .space  .up  .down .left .right `
如果想要指定的某一个某个按键的话 就必须重新的定义 找[键盘码](https://www.cnblogs.com/yiven/p/7118056.html) 
* 定义全局键盘码
 
```
 Vue.config.keyCodes.f2=113
```

 * `@click="add(传入参数) `
*  `v-text`在插值表达式中，不能使用`{item，Ctime} `只能这样使用使用  `v-text="item.Ctime`
#### 过滤器
*  调用就近原则，如果全局过滤器和私有过滤器名称一样，作用一样的，那么先调用私有的
* 过滤器调用的格式  `{ name | nameope }  `,过滤器可以多次调用
* 要想使全局过滤器生效的话，就必须初始化在 全局过滤器之后

```
     //   全局过滤器，所有的实例对象都共享了 
        // 其实这个时间会默认的给与他  pattern="" 以防止调用者不给他赋值
        Vue.filter("dateFromat", function (date, pattern = "") {
            var dt = new Date(date);
            var y = dt.getFullYear();
            var m = dt.getMonth() + 1
            var d = dt.getDate()
            console.log("shiming" + pattern)
            console.log("shiming" + date)
            // return `${y}-${m}-${d}`

            // pattern 这个要判断一下是否为null 
            if (pattern.toLowerCase() === "yyyy-mm-dd") {
                // return y+"-"+m+"-"+d
                // 魔法字符串
                return `${y}-${m}-${d}`
            } else {
                var hh = dt.getHours();
                var mm = dt.getMinutes()
                var ss = dt.getSeconds()

                return `${y}-${m}-${d} ${hh}:${mm}:${ss}`
            }
        })
```

* forEach some filter findIndex 都会遍历数组的每一项，
  *  forEach没有办法终止 
  *  some 可以return true 把它终止掉
  *  filter 
  *  findIndex 找索引 


####  一个添加和删除数据的Demo

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
    <!-- 这个是干嘛的？？？ -->
    <link rel="stylesheet" href="./lib/bootstrap.min.css">
</head>

<body>
    <div id="app">


        <div class="panel panel-primary">
            <div class="panel-heading">
                <h3 class="panel-title">添加品牌</h3>
            </div>
            <div class="panel-body form-inline">
                <label>
                    Id:<input type="text" class="form-control" v-model="id">
                </label>
                <label>
                    <!-- class="form-control" 一个好看一个不好看  @keyup="add" 事件-->
                    <!--  @keyup.enter="add" 点击了回车键的话，就触发事件 
                    全部的按键别名
                    .enter  .tab .delete  .esc  .space  .up  .down .left .right 
                    如果想要指定的某一个某个按键的话 就必须重新的定义 找键盘码 
                    -->
                    <!-- Name:<input type="text" v-model="name" @keyup.enter="add"> -->
                    <!-- 这样就可以其作用了 使用f2 去触发某些的操作 -->
                    Name:<input type="text" v-model="name" @keyup.f2="add">
            
                </label>
                <!-- 传入参数 ，加了小括号  -->
                <input type="button" value="添加" class="btn btn-primary" @click="add()">


                <label>
                    搜索关键字:<input type="text" class="form-control" v-model="key">
                </label>
            </div>
        </div>


        <table class="table table-bordered table-hover table-striped">
            <thead>
                <tr>
                    <th> ID</th>
                    <th> Name</th>
                    <th> Ctime</th>
                    <th> Operation</th>


                </tr>
            </thead>
            <tbody>
                <!-- 如果这个list是个固定的集合的话，那么页面就不能刷新了，我个人的理解
                这个list需要根据代码来改变的，这样子才很好  
                -->
                <tr v-for="item in search(key)" :key="item.id">
                    <td>{item.id}</td>
                    <td>{item.name}</td>
                    <!-- v-text在插值表达式中，不能使用{{item，Ctime}} 只能这样使用使用 
                         <td v-text="item.Ctime"></td>

                          这种用法是错误的，下面的使用的方法    <td v-text="{{item.Ctime}}"></td>
                    -->
                    <!-- <td v-text="item.Ctime"></td> -->

                    <!-- <td>{{item.Ctime | msgFormat('你好仕明同学')|test}}</td> -->

                    <td>{{ item.Ctime | dateFromat("yyyy-mmd-dd") }}</td>
                    <td>
                        <!-- .prevent 阻止默认行为 -->
                        <a href="" @click.prevent="del(item.id)">删除</a>
                    </td>
                </tr>
            </tbody>
        </table>


    </div>


    <div id="app2">


        {{1+1}}

        {{ dt |dateFromat }}

        <h1> {{ dt |dateFromat }} </h1>
    </div>


    <div id="app3">
        {{1+1}}

    </div>
    <script>




        // Vue.filter("msgFormat", function (msg, args) {
        //     return msg + "arg" + args
        // })
        // // 过滤器可以多次调用

        // Vue.filter("test", function (msg) {
        //     return msg + "test" 
        // })
        //   全局过滤器，所有的实例对象都共享了 
        // 其实这个时间会默认的给与他  pattern="" 以防止调用者不给他赋值
        Vue.filter("dateFromat", function (date, pattern = "") {
            var dt = new Date(date);
            var y = dt.getFullYear();
            var m = dt.getMonth() + 1
            var d = dt.getDate()
            console.log("shiming" + pattern)
            console.log("shiming" + date)
            // return `${y}-${m}-${d}`

            // pattern 这个要判断一下是否为null 
            if (pattern.toLowerCase() === "yyyy-mm-dd") {
                // return y+"-"+m+"-"+d
                // 魔法字符串
                return `${y}-${m}-${d}`
            } else {
                var hh = dt.getHours();
                var mm = dt.getMinutes()
                var ss = dt.getSeconds()

                return `${y}-${m}-${d} ${hh}:${mm}:${ss}`
            }
        })

        // 定义全局键盘码 
        Vue.config.keyCodes.f2=113

      
        //  要想使全局过滤器生效的话，就必须初始化在 全局过滤器之后
        var vm2 = new Vue({
            el: "#app2",
            data: {
                dt: new Date()
            },
            methods: {

            },
            // 定义私有过滤器
            filters: {

                // 调用就近原则，如果全局过滤器和私有过滤器名称一样，作用一样的，
                // 那么先调用私有的
                dateFromat: function (date, pattern = "") {
                    var dt = new Date(date);
                    var y = dt.getFullYear();
                    // 有可能是一位数的  padStart(2,"0") 表示长度为2 ，不够的话0来补充
                    var m = (dt.getMonth() + 1).toString().padStart(2,"0")
                    var d = dt.getDate().toString().padStart(2,"0")
                    console.log("shiming" + pattern)
                    console.log("shiming" + date)
                    // return `${y}-${m}-${d}`

                    // pattern 这个要判断一下是否为null 
                    if (pattern.toLowerCase() === "yyyy-mm-dd") {
                        // return y+"-"+m+"-"+d
                        // 魔法字符串
                        return `${y}-${m}-${d}`
                    } else {
                        var hh = dt.getHours().toString().padStart(2,"0");
                        var mm = dt.getMinutes().toString().padStart(2,"0")
                        var ss = dt.getSeconds().toString().padStart(2,"0")

                        return `${y}-${m}-${d} ${hh}:${mm}:${ss}`+"私有的过滤器"
                    }
                }
            }
        })




        var vm = new Vue({
            el: "#app",
            data: {
                id: "",
                name: "",
                key: "",
                list: [
                    { id: 1, name: "宝马", Ctime: new Date() },
                    { id: 2, name: "长安", Ctime: new Date() },
                ]
            },
            methods: {
                add() {
                    console.log("log")

                    var car = { id: this.id, name: this.name, Ctime: new Date() }

                    this.list.push(car)
                    this.name = this.id = ""

                },
                del(id) {
                    //vue中使用some删除list中的数据 可以在some内部做任何的事情 
                    //    this.list.some((item,i)=>{
                    //        if(item.id==id){
                    //            this.list.splice(i,1)
                    //            return true; 
                    //        }
                    //    })  
                    //这个就是专门来找id的 
                    var index = this.list.findIndex(item => {
                        if (item.id == id) {

                            return true;
                        }
                    })
                    this.list.splice(index, 1)
                    console.log("找到的索引 :" + index)
                },
                //根据关键字查询集合
                search(key) {
                    //方法一
                    // var newList=[];
                    // // 这里这个循环注意是使用的那个item，是条件的哦
                    // this.list.forEach(item => {
                    //     if (item.name.indexOf(key)!=-1) {
                    //         newList.push(item)
                    //     }
                    // }); 
                    // return newList

                    //  forEach some filter findIndex 都会遍历数组的每一项，
                    // forEach没有办法终止 
                    // some 可以return true 把它终止掉
                    // filter 
                    // findIndex 找索引 
                    //过滤器，符合的都返回
                    return this.list.filter(item => {
                        // 如果能取到的话，就不等于-1 
                        if (item.name.indexOf(key) != -1) {

                        }
                        // es6中提供新的方法，叫做includes 如果包含返回true
                        if (item.name.includes(key)) {
                            return item
                        }
                    })
                    //  newList
                }

            }
        })
        // 过滤器的定义方法 主要去解决时间显示的问题
        // Vue.filter("过滤器的名称",function(data){
        //      return data+"123"
        // })


    </script>
</body>

</html>


<!-- 过滤器调用的格式  { name | nameope }-->
```


* 自定义指令 在Vue中所有的指令都是` V`开头`<input type="text" class="form-control" v-model="key" v-focus>`
* 使用`Vue.directive` 定义全局的指令
   * 参数1：指令的名称，注意不需要加上V- ，但是调用的时候，需要加上V-
   * 参数2：是一个对象，有指令相关的函数，函数可以在特点的阶段执行相关的操作
  
```
// 使用Vue.directive 定义全局的指令
// 参数1：指令的名称，注意不需要加上V- ，但是调用的时候，需要加上V-
// 参数2：是一个对象，有指令相关的函数，函数可以在特点的阶段执行相关的操作
         Vue.directive("focus",{
            //  指令绑定在元素上的时候，只调用一次，在指令第一次绑定到元素上时调用。就是一个元素的对象
             bind:function(el){
                // 每个函数中，第一个参数永远是el，表示被绑定，其实就是text 原声的js对象
                // 一个元素，只有插入Dom之后，才会有焦点
                // el.focus() 所以这个方法在这里不行
             },
            //  inserted 表示元素插入Dom中的时候，会执行一次
             inserted:function(el){
                el.focus()
             },
               // 值更新时的工作
             // 也会以初始值为参数调用一次
             updated:function(){

             }
         })
```

* #### [钩子函数参数](https://cn.vuejs.org/v2/guide/custom-directive.html#%E9%92%A9%E5%AD%90%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0 "钩子函数参数")
*    `<input type="text" v-focus V-color="'blue'"    >`
```

         Vue.directive("color",{
            //  为啥设置颜色可以生效，其实理解为初始化了这个属性
            // 只要通过指令绑定了元素，不管这个元素有没有插入进去，这个元素肯定有一个内联的样式
            // 和样式相关的操作 
             bind:function(el,binding){
                  el.style.color="red"
//                   binding：一个对象，包含以下属性：
// name：指令名，不包括 v- 前缀。
// value：指令的绑定值，例如：v-my-directive="1 + 1" 中，绑定值为 2。
// oldValue：指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。
// expression：字符串形式的指令表达式。例如 v-my-directive="1 + 1" 中，表达式为 "1 + 1"。
// arg：传给指令的参数，可选。例如 v-my-directive:foo 中，参数为 "foo"。
// modifiers：一个包含修饰符的对象。例如：v-my-directive.foo.bar 中，修饰符对象为 { foo: true, bar: true }。
                  console.log("shiming value=---"+binding.value)  
                  console.log("shiming name=---"+binding.name)       
                  console.log("shiming expression=---"+binding.expression) 

                         el.style.color=binding.value   
             },
            //  和行为有关的操作，最好在inserted中执行
             inserted:function(){

             },
             updated:function(){

             }
         })
```

* 定义私有的指令，写到vm实例中去了

```
  // 定义私有的指令
            directives:{
                "fontweight":{
                    bind:function(el,binding){
                        el.style.fontWeight=binding.value
                    }
                }
            }
```

* 注意`directives` ，引用的时候是大写的V `<h1 v-color="'pink'" V-fontweight="900"> { dt |dateFromat } </h1>`

* 函数简写
  * 在很多时候，你可能想在 bind 和 update 时触发相同行为，而不关心其它的钩子。
  
```
   // 定义私有的指令
            directives:{
                "fontweight":{
                    bind:function(el,binding){
                        el.style.fontWeight=binding.value
                    }
                },
                // 这个方法其实就两个方法合成一个了，bind和update中去了
                "fontsize" :function(el,binding){
                   el.style.fontSize=binding.value
                }

            }
```

#### Vue的生命周期
* 1、`new Vue（） `
* 2、 `Init` ：刚刚初始化了一个`Vue`实例，只有默认的生命周期函数和默认的事件，其他的没有创建
* 3、 实例完全被创建出来，会执行他，在这个生命周期函数执行的时候，`data`和`methods`的还没有被初始化
* 4、初始化`data`和`methods `
* 5、`created` 方法执行，`data`和`methods` 初始化好了
* 6、`Vue `开始编译模板，`Vue`代码中的指令进行执行，最终在内存中生成一个编译好的最终模板，然后把这个模板字符串，渲染为内存中的`Dom`，此时，只是在内存中，渲染好了模板，并没有把模板挂载到正真的页面中去
* 7、内存中已经已经编译完成了，但是没有把模板渲染到页面中`beforeMount()`,` console.log(document.getElementById("pp").innerText)`就是 输出 ` innerText {msg` 还没有正真的渲染出来，还咩有挂载到页面中去
* 8、强内存中编译好的模板，正式的替换到页面中去
* 9、内存中的模板挂载到页面中，用户可以看到页面的 `mounted`实例最后的一个生命周期的函数只要执行完了`mounted`，`Vue`实例已经初始化完毕了，此时已经进入到了运行阶段
* 10、`beforeUpdate`  最少执行0次，也有可能触发多次，运行中的事件
* 11、 `update` 最少执行0次，也有可能触发多次，运行中的事件
* 12、`beforeDestory `Vue实例就已经从运行阶段进入到销毁阶段，但是此时候实例上所有`data`和`methods`，过滤器和指令都是处于可用的状态，还没有执行销毁的过程
* 13、`destoryed`到这里就全部销毁了
![lifecycle.png](https://upload-images.jianshu.io/upload_images/5363507-ad17631a27fc1f09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* Demo 代码

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
</head>

<body>

    <div id="app">
        <p>好好学习</p>

        <p id="pp">{{ msg }}</p>
        <input type="button" value="修改msg" @click="msg='我被修改了'">
    </div>


    <script>
        // 1、new Vue
        //2、 init:刚刚初始化了一个Vue实例，只有默认的生命周期函数和默认的事件，其他的没有创建

        var vm = new Vue({
            el: "#app",
            // 4、初始化data和methods 
            data: {
                msg: "msg消息"
            },
            methods: {
                show() {
                    console.log("show方法")
                }
            },
            //3、 实例完全被创建出来，会执行他，在这个生命周期函数执行的时候，data和methods的还没有被初始化
            beforeCreate() {
                console.log("beforeCreate方法")
                // 输出为undefined
                console.log(this.msg)
            },
            // 5、created 方法执行，data和methods 初始化好了
            created() {
                console.log("created方法")
                console.log(this.msg)
            },
            // 6、Vue 开始编译模板，Vue代码中的指令进行执行，最终在内存中生成一个编译好的最终模板，
            // 然后把这个模板字符串，渲染为内存中的Dom，此时，只是在内存中，渲染好了模板，并没有把模板挂载到正真的页面中去
            //   7、内存中已经已经编译完成了，但是没有把模板渲染到页面中
            beforeMount() {
                console.log("beforeMount方法")
                // 就是 输出  innerText {{msg}} 还没有正真的渲染出来，还咩有挂载到页面中去
                console.log(document.getElementById("pp").innerText)
            },
            // 8、强内存中编译好的模板，正式的替换到页面中去
            // 9、内存中的模板挂载到页面中，用户可以看到页面的 mounted实例最后的一个生命周期的函数
            // 只要执行完了mounted，Vue实例已经初始化完毕了，此时已经进入到了运行阶段
            mounted(){
                console.log("mounted方法")
                console.log(document.getElementById("pp").innerText)
            },
            // 10、beforeUpdate  最少执行0次，也有可能触发多次，运行中的事件
            beforeUpdate(){
                console.log("beforeUpdate方法")
                // 这个消息还是旧的
                console.log(document.getElementById("pp").innerText)
            } ,
               // 11、 update 最少执行0次，也有可能触发多次，运行中的事件
            updated(){
                console.log("updated方法")
                console.log(document.getElementById("pp").innerText)
            },
            // 12、beforeDestory Vue实例就已经从运行阶段进入到销毁阶段，但是此时候实例上所有data和methods，过滤器和指令都是处于可用的状态，还没有执行销毁的过程

            beforeDestroy(){
                console.log("beforeDestroy方法")
            },
            // 13、到这里就全部销毁了
            destoryed(){
                console.log("destoryed方法")
            }   

        })

    </script>
</body>

</html>
```

#### vue-resource的使用

*  依赖于 `Vue `有个先后的顺序
*  ` <script src="https://cdn.jsdelivr.net/npm/vue-resource@1.5.1"></script>`
```
    <script src="./lib/vue.min.js"></script>
    <!-- vue-resource 依赖于 Vue 有个先后的顺序  this.$http.get -->
    <script src="https://cdn.jsdelivr.net/npm/vue-resource@1.5.1"></script>

// global Vue object
Vue.http.get('/someUrl', [config]).then(successCallback, errorCallback);
Vue.http.post('/someUrl', [body], [config]).then(successCallback, errorCallback);

// in a Vue instance
this.$http.get('/someUrl', [config]).then(successCallback, errorCallback);
this.$http.post('/someUrl', [body], [config]).then(successCallback, errorCallback);
```
* Demo
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
    <!-- vue-resource 依赖于 Vue 有个先后的顺序  this.$http.get -->
    <script src="https://cdn.jsdelivr.net/npm/vue-resource@1.5.1"></script>
</head>

<body>
    <div id="app">
        <input type="button" value="get" @click="getInfo">
    </div>
    <script>
        var vm = new Vue({
            el: "#app",
            data: {

            },
            methods: {
                getInfo() {
                    console.log("getInfo")


                    // GET /someUrl then返回的数据
                    this.$http.get('https://cn.vuejs.org/v2/guide/ssr.html').then(response => {


                        console.log(response)
                        // get body data  一般获取body 就可以了
                        this.someData = response.body;
                        console.log(this.someData)


                    }, response => {
                        // error callback
                        console.log(response)
                    });
                    
                    this.$http.post('http://vue.studyit.io/api/post',{},{emulateJSON:true}).then(result=>{
                        console.log(result.body)
                    })
                     

                    // // POST /someUrl
                    // this.$http.post('/someUrl', { foo: 'bar' }).then(response => {

                    //     // get status
                    //     response.status;

                    //     // get status text
                    //     response.statusText;

                    //     // get 'Expires' header
                    //     response.headers.get('Expires');

                    //     // get body data
                    //     this.someData = response.body;

                    // }, response => {
                    //     // error callback
                    //     console.log("error"+response)
                    // });
                }
            }

        })
    </script>
</body>

</html>
```
* 使用最多的请求`jsonp(url, [config])`
* [ vue-resource](https://github.com/pagekit/vue-resource/blob/develop/docs/http.md) 目前和Vue中最常用的请求框架



#### Jsonp的原理（还要仔细查阅资料）
* **jsonp的实现原理**
**1、由于浏览器的安全限制，不允许AJAX访问 协议不同，余名不同，端口号不同的数据接口，浏览器认为不安全**
**2、可以通过动态创建script的标签的形式，把script标签的src属性，指向数据接口的地址，因为script标签不存在跨域限制，这种方式叫做JSONP，JSONP只支持get请求**

* **自己的`node`服务器**
* `node.js`服务器的实现个人感觉好像`go`,我擦，这一套代码都是一个人写的？？
* [nodejs开发辅助工具nodemon](https://www.cnblogs.com/xiaohuochai/p/8794340.html)
```
// 导入http的内置模块
const http = require("http")
// 创建一个 http服务器
const server = http.createServer()
// 监听 http 服务器的request的请求
server.on('request', function (req, res) {

    const url = req.url

    console.log("我是服务器，我启动了url==="+url)

    if (url === "/getDemo") {

        var scrip = "show()"

        res.end(scrip)

    } else {
        // 返回404
        res.end("404")
    }
})
// http://127.0.0.1:3000/getDemo
// 指定端口号并启动服务器监听 感觉好像go语言啊
server.listen(3000, function () {
    console.log("我是服务器，我启动了")
})
```

* 启动
![image.png](https://upload-images.jianshu.io/upload_images/5363507-453921c3deafb876.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 客服端
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
     <script>
// jsonp的实现原理
// 1、由于浏览器的安全限制，不允许AJAX访问 协议不同，余名不同，端口号不同的数据接口，浏览器认为不安全
// 2、可以通过动态创建script的标签的形式，把script标签的src属性，指向数据接口的地址，因为script标签不存在跨域限制，这种方式叫做JSONP，JSONP只支持get请求
// 

         function show(){
             console.log("我是客服端的方法,但是通过服务器帮我执行的，show方法是服务器帮我调用的")
         }
        </script>

        <script src="http://127.0.0.1:3000/getDemo"></script>
</body>
</html>
```

* **后语：Vue+node.js来实现一个前端项目？**
