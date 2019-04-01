---
title: Vue.js学习No.4
date: 2019-04-01 09:21:03
tags: Vue
---
* **欢迎关注我的公众号**
![公众号](https://upload-images.jianshu.io/upload_images/5363507-0a0cf2e5fd8f843d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  --> 
* **学习的内容如下**


* **开始**
* **父组件向子组件传递值**
* 父组件，可以在引用子组件的时候，通过属性绑定（v-bind:）的 形式，把需要传递给子组件的数据，以属性绑定的形式，传递到子组件的内部 
  `       <com1 v-bind:parentmsg="msg"></com1>`
* 子组件不能访问到 父组件data中的数据，就是访问不到msg，但是有方法可以访问到
* 子组件中的data的数据，并不是通过父组件传递的，是自己私有的，
* 子组件通过Ajax请求回来的数据 
* data里面的数据是可读可写的

*  把父组件传递过来的属性parentmsg，定义一下，这样才能使用父组件中的数据
```
   template:"<h1 @click='change'>子组件  ---{{parentmsg}}--</h1>",
```
*  把父组件传递过来的属性parentmsg，定义一下，这样才能使用父组件中的数据
*  **props里面的数据是只读的，不能够更改的，但是好像现在可以了？？,而且程序不报错？？？？** 
```
                props:["parentmsg"],
```

* 修改了 props里面的数据是只读的，不能够更改的，但是好像现在可以了？？
![image.png](https://upload-images.jianshu.io/upload_images/5363507-472363cee722e992.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* Demo父组件向子组件传递值
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
         <!-- 父组件，可以在引用子组件的时候，通过属性绑定（v-bind:）的
        形式，把需要传递给子组件的数据，以属性绑定的形式，传递到子组件的内部 -->
         <com1 v-bind:parentmsg="msg"></com1>

    </div>
    <script>
        //可以把vm当成一个组件
    var vm=new Vue({
        el:"#app",
        data() {
            return {
                 msg:"父组件---我是vue的数据"
            }
        },
        methods: {
            
        },
        components:{
            //子组件不能访问到 父组件data中的数据，就是访问不到msg，但是有方法可以访问到
            com1:{
                //子组件中的data的数据，并不是通过父组件传递的，是自己私有的，
                // 子组件通过Ajax请求回来的数据 
                // data里面的数据是可读可写的
               data() {
                   return {
                       title:"dd"
                   }
               },

                template:"<h1 @click='change'>子组件  ---{{parentmsg}}--</h1>",
                //  把父组件传递过来的属性parentmsg，定义一下，这样才能使用父组件中的数据
                // props里面的数据是只读的，不能够更改的，但是好像现在可以了？？
                props:["parentmsg"],
                methods: {
                    change(){
                        this.parentmsg="修改了 props里面的数据是只读的，不能够更改的，但是好像现在可以了？？"
                    }
                },
            }
        }
    })
    </script>
</body>
</html>
```
* **子组件向父组件传递值**
* 子组件传递给父组件值，其实就是父组件传递一个方法，子组件调用方法，然后把值传递给父组件

* 事件绑定机制，v-on 把父组件的show的方法传递给子组件的event
*  `v-on `可以简写为 `@`符号  
```
        <com2 v-on:event="show"></com2>
```
* 通过制定了一个ID，去加载id的template元素中的内容，当做组件的HTML结构

 ```
template: "#tmpl", 
```
* 如何拿到父组件的方法
```
 // 如何拿到父组件的方法
                    //emit 愿意就是触发，调用
                    this.$emit("event", this.ddddd)
                    this.$emit("event", "我是子组件传递过来的值哦")
                    this.$emit("event", 123456)

```
* 触发父组件的方法
![image.png](https://upload-images.jianshu.io/upload_images/5363507-0edc4462e8a22127.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5363507-c89048579a7ac932.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **為什麼打印不出來這個對象？？？？？ 我已經延遲3s了啊啊啊啊啊,我想打印出来子组件传递给父组件的值，但是代码执行会报错？？？？我开头以为赋值有延迟，还延迟3s去执行，但是还是有问题？**
```
 setTimeout(function () {
                        console.log('执行了');
                        console.log(shimingForSon)
                    }, 3000);
```

* **Demo子组件向父组件传递值**
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
<!--  -->
<!-- 子组件传递给父组件值，其实就是父组件传递一个方法，子组件调用方法，然后把值传递给父组件 -->
<body>

    <div id="app">
        <!-- 事件绑定机制，v-on 把父组件的show的方法传递给子组件的event-->
        <!-- v-on 可以简写为 @符号  -->
        <com2 v-on:event="show"></com2>
    </div>

    <template id="tmpl">

        <div>
            <h2>我是tmpl组件</h2>
            <input type="button" value="点我，可以触发父组件的方法" @click="myDemo">
        </div>
    </template>
    <script>
        //定义了一个字面量类型的组件模板对象 
        var com2 = {
            // 通过制定了一个ID，去加载id的template元素中的内容，当做组件的HTML结构
            template: "#tmpl",
            data() {
                return {
                    // 对象
                    ddddd: { name: "仕明同学", age: "18" }
                }
            },
            methods: {
                myDemo() {
                    console.log("我是子组件")
                    // 如何拿到父组件的方法
                    //emit 愿意就是触发，调用
                    this.$emit("event", this.ddddd)
                    this.$emit("event", "我是子组件传递过来的值哦")
                    this.$emit("event", 123456)

                }
            },
        }

        //可以把vm当成一个组件
        var vm = new Vue({
            el: "#app",
            data() {
                console.log("data我开始执行了")
                return {
                    msg: "父组件---我是vue的数据",
                    shimingForSon: null
                }
            },
            methods: {
                // 这个data可以让子组件传递参数过来
                show(data) {
                    console.log("我是父组件的方法show" + data)
                    console.log(data)
                    this.shimingForSon = data
                    // 為什麼打印不出來這個對象？？？？？ 我已經延遲3s了啊啊啊啊啊
                     setTimeout(function () {
                        console.log('执行了');
                        console.log(shimingForSon)
                    }, 3000);
                    // console.log(shimingForSon)


                }
            },
            components: {
                com2
            }

        })
    </script>
</body>

</html>
```

*  **Demo 评论的列表**

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.min.js"></script>
    <!-- 注意是rel 啊 -->
    <link rel="stylesheet" href="./lib/bootstrap.min.css">
</head>

<body>
    <div id="app">
            <!-- loaddata的方法是父组件的传递给子组件  -->
        <box @func="loadData"></box>
        <ul class="list-group">
            <li class="list-group-item" v-for="item in list" :key="item.id">
                <span class="badge">{{item.user}}</span>
                {{item.content}}
            </li>

        </ul>



    </div>

    <template id="tmpl">

        <div>
            <div class="form-group">
                <label>人：</label>
                <textarea class="form-control" v-model="user"></textarea>
            </div>

            <div class="form-group">
                <label>内容：</label>
                <textarea class="form-control" v-model="content"></textarea>
            </div>
            <div class="form-group">
                <input type="button" value="发表" @click="add" class="btn btn-primary">
            </div>
        </div>
    </template>
    <script>
        var commentBox = {
            template: "#tmpl",
            data() {
                return {
                    user: "",
                    content: "",

                }
            },
            methods: {
                add() {
                    console.log("点击组件的")
                    // 数据存放到了 localStorage中去了 
                    // 组织一个最新的评论数据对选哪个
                    // 把对象存储到localStorage中去
                    // 注意 localStorage中支持存放字符串数据，先调用 JSON.stringify
                    // 注意  数据完整性，新加的数据要和旧的数据一起存储  
                    // 注意 如果数据不存在的话，就直接返回一个空的数组
                    // 注意单词千万不要写错了啊
                    var commentqq = {
                        id: Date.now(), user: this.user, content: this.content
                    }
                    // 从本地获取数据
                    var strdata = localStorage.getItem("cmts") || "[]"
                    var list = JSON.parse(strdata)
                    // 注意顺序
                    // list.push(commentqq)
                    // 就是把数据添加到首部 
                    list.unshift(commentqq)

                    //  保存数据到本地

                    localStorage.setItem("cmts", JSON.stringify(list))

                    // 清空数据
                    this.user = this.content = ""
                    console.log("点击组件的 end" + list + this.user)

                    // 点击完了 我需要刷新数据

                    // func  是父组件传递过来的方法  子组件来触发这个方法
                    this.$emit("func")
                }
            },
        }

        var vm = new Vue({
            el: "#app",
            data: {
                list: [
                    { id: Date.now(), user: "shiming", content: "text" },
                    { id: Date.now(), user: "shiming1", content: "text1" },
                    { id: Date.now(), user: "shiming2", content: "text2" }
                ]
            },
            methods: {
                loadData() {
                    var strdata = localStorage.getItem("cmts") || "[]"
                    var list = JSON.parse(strdata)
                    this.list = list
                }
            },
            //初始化好了
            created() {
                this.loadData()
            },
            components: {
                "box": commentBox
            }
        })
    </script>

</body>

</html>
```

* **`ref `获取DOM元素和组件的引用** 
*  说通俗一点就是可以使用子组建的方法和数据
*    `console.log(this.$refs.id_h3.innerText)    `
```
    <div  id="app">
         <input type="button" value="获取元素" @click="getElement" ref="btn">

<!-- ref 可以指定 原生的DOM对象 -->
         <h3 id="id_h3"  ref="id_h3">你好啊</h3>
    </div>
    
```
![image.png](https://upload-images.jianshu.io/upload_images/5363507-12fb5b541b4c507c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
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
</head>

<body>

    <div id="app">
        <input type="button" value="获取元素" @click="getElement" ref="btn">

        <!-- ref 可以指定 原生的DOM对象 -->
        <h3 id="id_h3" ref="id_h3">你好啊</h3>
        <hr>
        <!-- 组件也可以使用 -->
         <login ref="myLogin"></login>
        
    </div>
 

    <template id="login">
        <h1>登陆</h1>
    </template>
    <script>

        var login = {
            template: "#login",
            data() {
                return {
                    msg:"我是组件"
                }
            },    
            methods: {
                show(){
                    console.log("调用了子组件的方法")
                }
            },
        }
        var vm = new Vue({
            el: "#app",

            data: {

            },

            methods: {
                getElement() {
                    // 通过id获取DOM元素 ，但是Vue不推荐
                    // <h3 id="id_h3">你好啊</h3>
                    console.log(document.getElementById("id_h3"))


                    console.log("我是Vue中获取DOM元素的对象的方法")
                    //  输出你好  我擦 牛逼
                    // ref 是 reference 
                    console.log(this.$refs.id_h3.innerText)

                    // 获取子组件中的 data
                    console.log(this.$refs.myLogin.msg)

                    // 调用子组件的方法
                    this.$refs.myLogin.show()

                }
            },

            components:{
               "login":login
              
            }
        })
    </script>
</body>

</html>
```

#### 路由
* 后端的路由：所有的URL地址对应都是服务器上对应的资源
* 前端路由：对于单页面的应用程序来说，主要是通过  `URL`的`hash （#）`号来实现不同页面之间的切换，同时`hash`有一个特点，`http`的请求不会包含`ash`相关的内容，所以单页面的程序主要是通过`hash`实现
* 使用路由的方法
* 1、导入包
```
 <!-- vue -->
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <!-- vue路由 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

```
* 2、定义组件
```
  var login = {
            template: "<h1>我是登陆组件</h1>"
        }
        var register = {
            template: "<h1>我是注册组件</h1>"
        }
```
* 3、定义对象 注意routes的拼写，没有r
```
      const router = new VueRouter({
            // 表示路由匹配的规则  
            routes: [

                // path 表示监听那个路由连接的地址
                // component 百世路由前面匹配到的path，展示相对的应的那个组件
                // component 必须是一个组件的模板对象，不能是组件应用的名称

                //  指定默认的为登陆，但是不推荐啊 
                //  {path:"/",component:login},
                // redirect进入页面的时候，直接就去login组件的页面
                { path: "/", redirect: "/login" },
                { path: "/login", component: login },

                { path: "/register", component: register }

            ],
            // 默认值: "router-link-active"

            // 设置 链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。
        //    改成自己的想要的样式
            linkActiveClass:"myStyle"

        })
```
* 4、注册到vm中去
```
      // 将路由规则对象，注册到vm实例上，用来监听URL对象的地址的变化。然后展示相对应的组件
            router: router
```
* **注意事项**
*  redirect进入页面的时候，直接就去login组件的页面 `    { path: "/", redirect: "/login" },`
*   "router-link-active" 设置 链接激活时使用的` CSS `类名。默认值可以通过路由的构造选项` linkActiveClass` 来全局配置。 改成自己的想要的样式
`  linkActiveClass:"myStyle"`
* `path` 表示监听那个路由连接的地址
* `component` 必须是一个组件的模板对象，不能是组件应用的名称

*  **路由传递参数方式一**
* 使用`k to="/login?id=10&name=shiming"` 传递参数，
```
  <!-- router-link 默认渲染为a标签  tag指定渲染成什么标签-->
        <!-- 如果在路由中，使用查询字符串，给路由传递参数，则不需要修改路由规则的path属性 -->
        <router-link to="/login?id=10&name=shiming" tag="span">登陆</router-link>
   
```
* 在组件中的`created`方法中获取参数
```
   // 组件的生命周期
            created() {
                console.log("我是登陆组件的日志")
                console.log(this.$route)
                console.log(this.$route.query.id)
            },
```
* 获取值为
![image.png](https://upload-images.jianshu.io/upload_images/5363507-473384674ce284d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 然后显示在控件上
```
     // 通过这种的方式可以拿到路径中的值
        var login = {
            template: "<h1>我是登陆组件  {{$route.query.id }}----{{$route.query.name}}</h1>",
            // 组件的生命周期
            created() {
                console.log("我是登陆组件的日志")
                console.log(this.$route)
                console.log(this.$route.query.id)
            },
        }
```
* `router-link-active` 其中的值的意思:
```
   /* 实现路由的文字的颜色 */
        .router-link-active {
            color: red;
            font-weight: 800;
            /* 倾斜 */
            font-style: italic;
            /* 大小 */
            font-size: 100px;
            /* 下划线 */
            text-decoration: underline;
            /* 背景色 */
            background-color: goldenrod
        }
```
* 完整的Demo
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <!-- vue -->
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <!-- vue路由 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        /* 实现路由的文字的颜色 */
        .router-link-active {
            color: red;
            font-weight: 800;
            /* 倾斜 */
            font-style: italic;
            /* 大小 */
            font-size: 100px;
            /* 下划线 */
            text-decoration: underline;
            /* 背景色 */
            background-color: goldenrod
        }

        .myStyle {
            color: hotpink;
        }

        .v-enter,
        .v-leaver-to {
            opacity: 0;
            transform: translateX(100px);

        }

        .v-enter-active,
        .v-leaver-active {
            transition: all 0.5s ease;
        }
    </style>
</head>

<body>

    <div id="app">
        <!--不推荐这样使用 -->
        <!-- <a href="#/login">登陆</a>
        <a href="#/register">注册</a> -->

        <!-- router-link 默认渲染为a标签  tag指定渲染成什么标签-->
        <!-- 如果在路由中，使用查询字符串，给路由传递参数，则不需要修改路由规则的path属性 -->
        <router-link to="/login?id=10&name=shiming" tag="span">登陆</router-link>
        <router-link to="/register">注册</router-link>
        <!-- 这是 vue-router中提供的元素，专门用来站位使用 -->
        <!-- 包裹动画  mode="out-in"动画的加载的方式-->
        <transition mode="out-in">
            <router-view></router-view>
        </transition>



    </div>

    <script>
        // 通过这种的方式可以拿到路径中的值
        var login = {
            template: "<h1>我是登陆组件  {{$route.query.id }}----{{$route.query.name}}</h1>",
            // 组件的生命周期
            created() {
                console.log("我是登陆组件的日志")
                console.log(this.$route)
                console.log(this.$route.query.id)
            },
        }
        var register = {
            template: "<h1>我是注册组件</h1>"
        }
        const router = new VueRouter({
            // 表示路由匹配的规则  
            routes: [

                // path 表示监听那个路由连接的地址
                // component 百世路由前面匹配到的path，展示相对的应的那个组件
                // component 必须是一个组件的模板对象，不能是组件应用的名称

                //  指定默认的为登陆，但是不推荐啊 
                //  {path:"/",component:login},
                // redirect进入页面的时候，直接就去login组件的页面
                { path: "/", redirect: "/login" },
                { path: "/login", component: login },

                { path: "/register", component: register }

            ],
            // 默认值: "router-link-active"

            // 设置 链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。
            //    改成自己的想要的样式
            linkActiveClass: "myStyle"

        })


        var vm = new Vue({
            el: "#app",

            data: {

            },

            methods: {

            },
            // 将路由规则对象，注册到vm实例上，用来监听URL对象的地址的变化。然后展示相对应的组件
            //    如果属性名一样的话，可以直接写 router
            // router: router
            router
        })
    </script>
</body>

</html>
```
* **路由传递参数方式二**
* 1、传入参数
```
  <!-- router-link 默认渲染为a标签  tag指定渲染成什么标签-->
        <router-link to="/login/10/lishiming" tag="span">登陆</router-link>
```

* 2、router定义`  { path: "/login/:id/:name", component: login },`
```
    const router = new VueRouter({
            // 表示路由匹配的规则  
            routes: [

                // path 表示监听那个路由连接的地址
                // component 百世路由前面匹配到的path，展示相对的应的那个组件
                // component 必须是一个组件的模板对象，不能是组件应用的名称

                //  指定默认的为登陆，但是不推荐啊 
                //  {path:"/",component:login},
                // redirect进入页面的时候，直接就去login组件的页面
                { path: "/", redirect: "/login" },
                // 第二种传递参数的方法 
                { path: "/login/:id/:name", component: login },

                { path: "/register", component: register }

            ],
            // 默认值: "router-link-active"

            // 设置 链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。
            //    改成自己的想要的样式
            linkActiveClass: "myStyle"

        })
```

* 3、 `router`如果属性名一样的话，可以直接写 
```
  // 将路由规则对象，注册到vm实例上，用来监听URL对象的地址的变化。然后展示相对应的组件
            //    如果属性名一样的话，可以直接写 router
            // router: router
            router
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
    <!-- vue -->
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <!-- vue路由 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        /* 实现路由的文字的颜色 */
        .router-link-active {
            color: red;
            font-weight: 800;
            /* 倾斜 */
            font-style: italic;
            /* 大小 */
            font-size: 100px;
            /* 下划线 */
            text-decoration: underline;
            /* 背景色 */
            background-color: goldenrod
        }

        .myStyle {
            color: hotpink;
        }

        .v-enter,
        .v-leaver-to {
            opacity: 0;
            transform: translateX(100px);

        }

        .v-enter-active,
        .v-leaver-active {
            transition: all 0.5s ease;
        }
    </style>
</head>

<body>

    <div id="app">
        <!--不推荐这样使用 -->
        <!-- <a href="#/login">登陆</a>
        <a href="#/register">注册</a> -->

        <!-- router-link 默认渲染为a标签  tag指定渲染成什么标签-->
        <router-link to="/login/10/lishiming" tag="span">登陆</router-link>
        <router-link to="/register">注册</router-link>
        <!-- 这是 vue-router中提供的元素，专门用来站位使用 -->
        <!-- 包裹动画  mode="out-in"动画的加载的方式-->
        <transition mode="out-in">
            <router-view></router-view>
        </transition>



    </div>

    <script>
        // 通过这种的方式可以拿到路径中的值
        var login = {
            template: "<h1>我是登陆组件  {{$route.params.id}}----{{$route.params.name}}</h1>",
            // 组件的生命周期
            created() {
                console.log("我是登陆组件的日志")
                console.log(this.$route.params.id)
            
            },
        }
        var register = {
            template: "<h1>我是注册组件</h1>"
        }
        const router = new VueRouter({
            // 表示路由匹配的规则  
            routes: [

                // path 表示监听那个路由连接的地址
                // component 百世路由前面匹配到的path，展示相对的应的那个组件
                // component 必须是一个组件的模板对象，不能是组件应用的名称

                //  指定默认的为登陆，但是不推荐啊 
                //  {path:"/",component:login},
                // redirect进入页面的时候，直接就去login组件的页面
                { path: "/", redirect: "/login" },
                // 第二种传递参数的方法 
                { path: "/login/:id/:name", component: login },

                { path: "/register", component: register }

            ],
            // 默认值: "router-link-active"

            // 设置 链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。
            //    改成自己的想要的样式
            linkActiveClass: "myStyle"

        })


        var vm = new Vue({
            el: "#app",

            data: {

            },

            methods: {

            },
            // 将路由规则对象，注册到vm实例上，用来监听URL对象的地址的变化。然后展示相对应的组件
            //    如果属性名一样的话，可以直接写 router
            // router: router
            router
        })
    </script>
</body>

</html>
```

* **路由的嵌套**
* 使用`children`的属性 实现路由的嵌套 
*  使用`children`属性，实现子路由，同时，子路由的`path`前面不要带 `/` ，如果带了，就永远以根路径开始请求，这样不方便我们用户去理解`URL`地址
* `  <router-view></router-view>`如果使用需要了使用就必须使用占位符
* Demo
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <!-- vue -->
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <!-- vue路由 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

</head>

<body>

    <div id="app">
        <!-- 这种方式也是可以引用组件的哦 -->
        <!-- <shiming></shiming> -->
        <router-link to="/acount">Acount</router-link>
        <router-view></router-view>

    </div>


    <template id="tmpl">

        <div>
            <h1>这是 Account组件 里面还嵌套了一个路由</h1>
            <!-- 路由的嵌套 -->
            <router-link to="/acount/login"> 登陆</router-link>

            <router-link to="/acount/register">注册</router-link>
            <!-- 占位符 必须要 -->
            <router-view></router-view>
        </div>
    </template>
    <script>

        var account = {
            template: "#tmpl"
        }
        var login = {
            template: "<h3>登陆</h3>"
        }
        var register = {
            template: "<h3>组成</h3>"
        }
        const router = new VueRouter({
            //  这里的不能写成 routers 哦
            routes: [
                {
                    path: "/acount",
                    component: account,
                    // 使用children的属性 实现路由的嵌套 
                    // 记住 ，login前面不能加上 / 
                    // 虽然还有另外的方式，但是呢？建议这样使用，然后方便去理解
                    // 使用children属性，实现子路由，同时，子路由的paht前面不要带 / ，如果带了，就永远以根路径开始请求，这样不方便我们用户去理解URL地址
                    children: [
                        { path: "login", component: login },
                        { path: "register", component: register }
                    ]
                },
                // // 
                //  { path: "/acount/login", component: login },

                //  { path: "/acount/register", component: register }
            ]
        })
        var vm = new Vue({
            el: "#app",

            data: {

            },

            methods: {

            },
            components: {
                "shiming": account
            },
            router
        })
    </script>
</body>

</html>
```

* **经典布局**
* `components` 对应很多的组件
* ` <router-view></router-view>`三个坑
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <!-- vue -->
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <!-- vue路由 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        /* 设置html页面中的 body边距  */
        html,body{
            margin: 0;
            padding: 0;
        }
        .container {
            display: flex;
        }
        /* h1 标签没有逗号哦 记住哦 */
        h1{
            margin: 0;
            padding: 0;
            font-size: 20px;
        }
        .header {
            background-color: aqua;
            height: 100px;
        }

        .left {
            background-color: yellow;
            flex: 2;
            height: 600px;
        }

        .main {
            background-color: blueviolet;
            flex: 10;
            height: 600px;
        }
    </style>
</head>

<body>

    <div id="app">

        <!-- 3个坑 -->
        <router-view></router-view>
        <div class="container">

            <router-view name="left"></router-view>

            <router-view name="main"></router-view>
        </div>

    </div>

    <script>

        var header = {
            template: "<h1 class='header'>Header 头部</h1>"
        }
        var leftBox = {
            template: "<h1 class='left'>left 侧边栏</h1>"
        }
        var mainBox = {
            template: "<h1 class='main'>mainBox 主体区域</h1>"
        }

        const router = new VueRouter({
            routes: [
                // 这种方法不能实现 
                // {path:"/",component:header},
                // {path:"/left",component:left},
                // {path:"/mainBox",component:mainBox},
                //  components 对应很多的组件
                {
                    path: "/", components: {
                        "default": header,
                        "left": leftBox,
                        "main": mainBox
                    }
                }
            ]
        })

        var vm = new Vue({
            el: "#app",

            data: {

            },

            methods: {

            },
            // router名字一样 可以简写哦
            router
        })
    </script>
</body>

</html>
```

* 以上，待续


